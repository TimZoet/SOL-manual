Framebuffer
===========

The :code:`sol::VulkanFramebuffer` class manages the lifetime of a :code:`VkFramebuffer`. A framebuffer is created
with a renderpass and requires a list of attachments matching what's in that renderpass, as well as the desired 
dimensions:

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::VulkanRenderPass& renderPass    = ...;
    sol::VulkanImageView& colorImageView = ...;
    sol::VulkanImageView& depthImageView = ...;

    // Creating a 1920x1080x1 framebuffer with a color and depth attachment.
    sol::VulkanFramebuffer::Settings settings;
    settings.renderPass = renderPass;
    settings.width      = 1920;
    settings.height     = 1080;
    settings.layers     = 1;
    settings.attachments.emplace_back(&colorImageView);
    settings.attachments.emplace_back(&depthImageView);

    auto framebuffer = sol::VulkanFramebuffer::create(setting);
