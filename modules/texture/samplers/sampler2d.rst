Sampler2D
=========

The :code:`sol::Sampler2D` class manages a 2-dimensional sampler object. Internally it holds an instance of
:code:`sol::VulkanSampler`, which manages the actual Vulkan sampler resource handle.

A new instance can be created through the static :code:`create` method:

.. code-block:: cpp
    :caption: Creating a sampler with nearest for filtering mode and repeat for addressing.

    sol::VulkanDevice& device = ...;

    sol::Sampler2DPtr sampler = sol::Sampler2D::create(sol::Sampler2D::Settings{
        .device = device,
        .magFilter = VK_FILTER_NEAREST,
        .minFilter = VK_FILTER_NEAREST,
        .mipmapMode = VK_SAMPLER_MIPMAP_MODE_NEAREST,
        .addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT,
        .addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT,
        .addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT
    });
