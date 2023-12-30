Image2D
=======

The :code:`sol::Image2D` class manages a 2-dimensional, single layer image. Internally it holds an instance of
:code:`sol::VulkanImage`, which manages the actual Vulkan image resource handle.

A new instance can be created through the static :code:`create` method. This allocates a new image with uninitialized
data:

.. code-block:: cpp
    :caption: Creating a 512x512 RGBA image for sampling in a fragment shader.

    sol::MemoryManager&                 manager = ...;
    sol::TransactionManager& transactionManager = ...;

    sol::Image2DPtr image = sol::Image2D::create(sol::Image2D::Settings{
        .memoryManager = manager,
        // 512x512 pixels.
        .size = {512u, 512u},
        // 8 bits per channel, srgb.
        .format = VK_FORMAT_R8G8B8A8_SRGB,
        // No mipmaps.
        .levels = 1,
        // We must also enable TRANSFER_DST, otherwise we cannot upload data.
        .usage = VK_IMAGE_USAGE_SAMPLED_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT,
        // This is a color image.
        .aspect = VK_IMAGE_ASPECT_COLOR_BIT,
        // Don't care about initial layout, that will be handled when uploading data.
        .initialLayout = VK_IMAGE_LAYOUT_UNDEFINED,
        // Place image on the graphics queue.
        .initialOwner = manager.getGraphicsQueue().getFamily(),
        // Unless we are on an integrated GPU, every image should use optimal tiling.
        .tiling = VK_IMAGE_TILING_OPTIMAL
    });

Setting the image data is done through the :code:`setData` method. In collaboration with a :code:`sol::Transaction` the
image object will create a staging buffer for transferring the specified data. This also requires a barrier to
synchronize access before and after the copy. Note that transitioning to the transfer layout is all handled by the
transaction:

.. code-block:: cpp
    :caption: Filling the image with some data.

    // Generate a pure red image with full opacity.
    const std::vector<uint32_t> data(image.getWidth() * image.getHeight(), (255u << 24) + 255u);
    
    // The barrier will be inserted around the copy commands.
    // Transition to the transfer layout is taken care of for us.
    constexpr sol::Image2D::Barrier barrier{
        // Already set to graphics on creation.
        .dstFamily = nullptr,
        // There were no previous commands yet.
        .srcStage = VK_PIPELINE_STAGE_2_NONE_KHR,
        // Fragment shader will be reading from the image.
        .dstStage = VK_PIPELINE_STAGE_2_FRAGMENT_SHADER_BIT,
        // There were no previous commands yet.
        .srcAccess = VK_ACCESS_2_NONE,
        // Fragment shader will be reading from the image.
        .dstAccess = VK_ACCESS_2_SHADER_READ_BIT,
        .dstLayout = VK_IMAGE_LAYOUT_READ_ONLY_OPTIMAL
    };

    // We copy the whole image in one go. If needed (e.g. for mipmaps)
    // you can specify multiple non-overlapping regions of the image.
    constexpr sol::Image2D::CopyRegion region{
        .dataOffset = 0,
        .level = 0,
        .regionOffset = {0, 0},
        .regionSize = {512, 512}
    };

    auto transaction = manager.beginTransaction();
    
    // Because allocation of the staging buffer might fail,
    // you should always check the return value of setData.
    if (!image->setData(*transaction,
                       data.data(),
                       sizeof(uint32_t) * data.size(),
                       barrier,
                       false,
                       {region})
        )
        throw std::runtime_error("Staging buffer allocation failed.")
    
    // Commit to perform the lone copy.
    transaction->commit();
    transaction->wait();

    // We can now safely use the image.

.. note:: 

    TODO: Examples of subregions and mipmaps.
