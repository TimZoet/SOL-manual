Image2D
=======

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    image2d/atlases
    image2d/compression
    image2d/mipmaps
    image2d/resizing

The :code:`sol::Image2D` class manages a 2-dimensional, single layer image. The object itself is owned by the 
:code:`sol::TextureManager`. Internally it holds an instance of :code:`sol::VulkanImage`, which manages the actual 
Vulkan image resource handle.

New images can be created through the :code:`sol::TextureManager`. There are various methods to create 
image objects with a default fill color, from a data array, not yet initialized, etc. At minimum, the format and 
dimensions of an image must be specified on construction:

.. code-block:: cpp

    sol::TextureManager& texManager = ...;

    // Create an image with uninitialized data.
    auto& image0 = texManager.createImage2D(VK_FORMAT_R8G8B8A8_SRGB, {512, 512});

    // Create an image filled with all red values. Automatically creates and fills a staging buffer.
    std::array<uint8_t, 4> rgba = {255, 0, 0, 255};
    auto& image1 = texManager.createImage2DFill(VK_FORMAT_R8G8B8A8_SRGB, {256, 256}, rgba.data());

    // Invoke the TextureManager to actually transfer the staging buffers.
    texManager.transfer();

Note that the actual buffer of either image is not updated until the :code:`sol::TextureManager` has transferred all
staged image data to the GPU.

By default it is assumed the image will be used as a transfer destination and is going to be sampled from. I.e., the
image usage flags are set to :code:`VK_IMAGE_USAGE_SAMPLED_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT`. This can be overriden
when constructing a new image:

.. code-block:: cpp

    // A typical image that might be written to in a compute shader
    // after which the data is transferred to main memory.
    texManager.createImage2D(
        VK_FORMAT_R8G8B8A8_UINT,
        {1024, 1024},
        VK_IMAGE_USAGE_STORAGE_BIT | VK_IMAGE_USAGE_TRANSFER_SRC_BIT
    );

Staging Buffers
---------------

You can create one or more staging buffers to update (parts of) the image. When using multiple staging buffers, they
should not overlap.

Staging buffers contain persistenly mapped memory. With a staging buffer created, you can repeatedly call the various
:code:`setData` and :code:`fill` methods to copy data to a particular staging buffer such that it can be transferred to
the actual image buffer by the manager:

.. code-block:: cpp

    // Assuming vector with enough data to update entire image.
    std::vector<uint8_t> colorData = {...};

    // Create staging buffer covering whole image and fill it.
    image0.createStagingBuffer();
    image0.setData(colorData.data(), 0);

    // Assuming someArray that contains non-overlapping chunks of data,
    // create and fill a distinct staging buffer for each chunk.
    for (auto [data, x, y, width, height] : someArray)
        const auto bufferIdx = image1.createStagingBuffer({x, y}, {width, height});
        image1.setData(data, {x, y}, {width, height}, bufferIdx);

    // Invoke the TextureManager to actually transfer the staging buffers.
    texManager.transfer();

Queue Family Ownership
----------------------

Images are created with exclusive ownership (:code:`VK_SHARING_MODE_EXCLUSIVE`). The :code:`sol::TextureManager` takes
care of ownership transfer. On creation, the image's current and target queue family are set to the graphics queue,
since that is where most images will be used. If the image is created with uninitialized data, no ownership transfer is
staged. The target family can be explicitly specified using the :code:`setTargetFamily` method. This can be done just
after creating it, but also at a later stage:

.. code-block:: cpp

    // image0 is created without data and target family is not explicitly changed.
    // No ownership transfer is staged.
    auto& image0 = texManager.createImage2D(VK_FORMAT_R8G8B8A8_SRGB, {512, 512});

    // image1 is also created without data...
    auto& image1 = texManager.createImage2D(VK_FORMAT_R8G8B8A8_SRGB, {512, 512});
    // ...but it is going to be used in a compute shader
    // and therefore needs to be owned by the compute queue.
    image1.setTargetFamily(memoryManager->getComputeQueue().getFamily());

    // Invoke the TextureManager to actually transfer ownership.
    texManager.transfer();

You should never modify the current family yourself, as this is handled by the :code:`sol::TextureManager`. Obviously, 
you should also not modify the target family while the manager is executing the transfer operations.

Image Layout Transitions
------------------------

All images start in an undefined layout (i.e. :code:`VK_IMAGE_LAYOUT_UNDEFINED`). Their target layout is set to
:code:`VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL`, since most use cases involve sampling images in shaders. Just as with
queue ownership, the target layout can be modified directly after creation and at a later stage:

.. code-block:: cpp

    // We are creating a depth image, perhaps to be used in conjunction with a swapchain image.
    // We should at the very least override the target layout, because the default is very wrong.
    auto& depthImage = texManager.createImage2D(
        VK_FORMAT_D32_SFLOAT,
        {1920, 1080},
        VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT
    );
    depthImage.setTargetLayout(VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL);
    depthImage.setAspectFlags(VK_IMAGE_ASPECT_DEPTH_BIT);

    // Perform image layout transitions.
    texManager.transfer();

    // Let's pretend a whole bunch of stuff happens here.
    ...

    // We're going to be transferring the depth image to main memory. For that,
    // it first needs to be in the right layout. (In practice you should probably
    // be doing this through your render passes, but this is just an example.)
    depthImage.setTargetLayout(VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL);
    texManager.transfer();

Stage and Access Flags
----------------------

By default it is assumed that the image will be sampled in the fragment shader. This may not always be true, in which
case you must modify the stage and access flags of the image. These flags are used during buffer transfer to synchronize
command execution and memory access. Setting the required values can be done with the :code:`setStageFlags` and 
:code:`setAccessFlags` methods:

.. code-block:: cpp

    // image0 is sampled in both vertex and fragment shader.
    image0.setStageFlags(VK_PIPELINE_STAGE_VERTEX_SHADER_BIT |
                         VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT);

    // image1 is written to in the compute shader.
    image1.setStageFlags(VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT);
    image1.setAccessFlags(VK_ACCESS_SHADER_WRITE_BIT);

Obviously, you should limit the flags to what is necessary for optimal performance.