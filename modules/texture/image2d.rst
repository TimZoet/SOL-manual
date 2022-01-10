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

    // Create an image with uninitialized data.
    auto& image0 = texManager.createImage2D(VK_FORMAT_R8G8B8A8_SRGB, {512, 512});

    // Create an image filled with all red values. Automatically creates and fills a staging buffer.
    std::array<uint8_t, 4> rgba = {255, 0, 0, 255};
    auto& image1 = texManager.createImage2DFill(VK_FORMAT_R8G8B8A8_SRGB, {256, 256}, rgba.data());

    // Invoke the TextureManager to actually transfer the staging buffers.
    ...

Note that the actual buffer of either image is not updated until the :code:`sol::TextureManager` has transferred all
staged image data to the GPU.

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
    ...

Queue Family Ownership
----------------------

Images are created with exclusive ownership (:code:`VK_SHARING_MODE_EXCLUSIVE`). The :code:`sol::TextureManager` takes
care of ownership transfer. On creation, the image's target queue family is set to the graphics queue, since that is 
where most images will be used. Should that not be the case, the target family can be modified using the
:code:`setTargetFamily` method:

.. code-block:: cpp

    // image1 is going to be used in the compute shader and
    // therefore needs to be owner by the compute queue.
    image1.setTargetFamily(memoryManager->getComputeQueue().getFamily());

.. note::

    The :code:`sol::TextureManager` does not yet support transferring just ownership without also copying data.

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