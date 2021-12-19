Image
=====

The :code:`sol::VulkanImage` class manages the lifetime of a :code:`VkImage`. Just as with the :code:`sol::VulkanBuffer`
it is possible to manage and bind memory yourself, or use the :code:`VulkanMemoryAllocator` instead.

Images have lots of properties that can be setting, ranging from format and dimensions to how they are to be shared
between queues:

.. code-block:: cpp
    
    // Assuming a previously created device.
    sol::VulkanDevice& device = ...;

    // First create an 8-bit RGBA image. It is going to be used for sampling and as a transfer destination.
    sol::VulkanImage::Settings settings;
    settings.device         = device;
    settings.format         = VK_FORMAT_R8G8B8A8_SRGB;
    settings.width          = 256;
    settings.height         = 256;
    settings.depth          = 1;
    settings.tiling         = VK_IMAGE_TILING_OPTIMAL;
    settings.imageUsage     = VK_IMAGE_USAGE_SAMPLED_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT;
    settings.sharingMode    = VK_SHARING_MODE_EXCLUSIVE;
    settings.initialLayout  = VK_IMAGE_LAYOUT_UNDEFINED;
    auto image              = sol::VulkanImage::create(settings);

    // Then allocate memory.
    sol::VulkanDeviceMemory::Settings memSettings;
    memSettings.device              = device;
    memSettings.size                = image->getMemoryRequirements().size;
    memSettings.memoryTypeBits      = image->getMemoryRequirements().memoryTypeBits;
    memSettings.memoryPropertyFlags = VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT;
    auto memory                     = sol::VulkanDeviceMemory::create(memSettings);

    // Finally, bind memory to image.
    image->bindMemory(memory);

Using the :code:`VulkanMemoryAllocator` follows the same pattern as for the :code:`sol::VulkanBuffer`.

Swapchain Images
----------------

When creating a :code:`sol::VulkanSwapchain`, the images that are created for it are also put inside of a 
:code:`sol::VulkanImage`. However, this is done without creating a fully specified :code:`Settings` object. Any values
other than the :code:`device` will be invalid. When an image is actually part of a swapchain, the 
:code:`isSwapchainImage` flag is set.

.. code-block:: cpp
    
    sol::VulkanSwapchain swapchain = ...;

    // Should be true.
    assert(swapchain.getImages().front()->getSettings().isSwapchainImage);

    // Do not access any other properties. They will not be valid.
    swapchain.getImages().front()->getSettings().width
