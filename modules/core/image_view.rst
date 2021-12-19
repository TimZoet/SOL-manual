Image View
==========

The :code:`sol::VulkanImageView` class manages the lifetime of a :code:`VkImageView`. Creation is simple:

.. code-block:: cpp
    
    // Assuming a previously created image.
    sol::VulkanImage::Settings settings;
    settings.device = device;
    settings.format = VK_FORMAT_R8G8B8A8_SRGB;
    ...
    auto image      = sol::VulkanImage::create(settings);

    // Create an image view with in this case the same format.
    sol::VulkanImageView::Settings settings;
    settings.image  = *image;
    settings.format = VK_FORMAT_R8G8B8A8_SRGB;
    settings.aspect = VK_IMAGE_ASPECT_COLOR_BIT;
    auto imageView  = sol::VulkanImageView::create(memSettings);
