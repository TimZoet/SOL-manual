Texture2D
=========

The :code:`sol::Texture2D` class can be used to specify how a :code:`sol::Image2D` should be sampled. Internally, it 
keeps a (non-owning) reference to a :code:`sol::Image2D`. Additionally, it holds a :code:`sol::VulkanImageView` and 
:code:`sol::VulkanSampler` to describe the sampling parameters. That way, multiple textures can refer to the same image
whilst being interpreted differently in shaders.

New textures can be created through the :code:`sol::TextureManager`. When passing just an image, a default image view 
and sampler are constructed:

.. code-block:: cpp

    // Assuming two previously created images.
    auto& image0 = texManager.createImage2D(...);
    auto& image1 = texManager.createImage2D(...);

    // Create a texture for each image.
    auto& tex0 = texManager.createTexture2D(image0);
    auto& tex1 = texManager.createTexture2D(image1);

It is also possible to provide the create method explicitly with a :code:`sol::VulkanImageView` and/or 
:code:`sol::VulkanSampler` object to override the defaults:

.. code-block:: cpp

    // Create another texture for the second image,
    // but pass a custom image view to swizzle the components.
    sol::VulkanImageView::Settings settings;
    settings.device     = ...;
    settings.image      = &image1.getImage();
    settings.format     = image1.getFormat();
    settings.aspect     = image1.getAspectFlags();
    settings.components = VkComponentMapping{
        VK_COMPONENT_SWIZZLE_B,
        VK_COMPONENT_SWIZZLE_G,
        VK_COMPONENT_SWIZZLE_R,
        VK_COMPONENT_SWIZZLE_A
    };
    auto imgView = sol::VulkanImageView::create(settings);
    auto&   tex2 = texManager.createTexture2D(image1, std::move(imgView));

Image Updates
-------------

.. note::
    Not yet implemented.

Sampler Updates
---------------

.. note::
    Not yet implemented.
