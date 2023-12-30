Texture2D
=========

The :code:`sol::Texture2D` class references a :doc:`/modules/texture/images/image2d` and
:doc:`/modules/texture/samplers/sampler2d`. Internally it holds an instance of :code:`sol::VulkanImageView`, which
defines how the referenced image is interpreted.

A new instance can be created through the static :code:`create` method:

.. code-block:: cpp
    :caption: Creating a texture from an image and sampler.

    sol::Image2D&     image = ...;
    sol::Sampler2D& sampler = ...;

    sol::Texture2DPtr texture = sol::Texture2D::create(sol::Texture2D::Settings{
        .image = image,
        .sampler = sampler,
        // Leave empty to just use image format and aspect.
        .format = {},
        .aspect = {},
        // Interpret as BGR, because why not.
        .components = {VK_COMPONENT_SWIZZLE_B, 
                       VK_COMPONENT_SWIZZLE_G,
                       VK_COMPONENT_SWIZZLE_R,
                       VK_COMPONENT_SWIZZLE_IDENTITY
                      }
    });
