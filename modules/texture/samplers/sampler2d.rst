Sampler2D
=========

The :code:`sol::Sampler2D` class manages a 2-dimensional sampler object. Internally it holds an instance of
:code:`sol::VulkanSampler`, which manages the actual Vulkan sampler resource handle.

A new instance can be created through the :code:`createSampler2D` method of the :code:`sol::TextureCollection`:

.. code-block:: cpp
    :caption: Creating a sampler with nearest for filtering mode and repeat for addressing.

    sol::TextureCollection&          collection = ...;

    sol::Sampler2D& sampler = collection.createSampler2D(
        VK_FILTER_NEAREST,
        VK_FILTER_NEAREST,
        VK_SAMPLER_MIPMAP_MODE_NEAREST,
        VK_SAMPLER_ADDRESS_MODE_REPEAT,
        VK_SAMPLER_ADDRESS_MODE_REPEAT,
        VK_SAMPLER_ADDRESS_MODE_REPEAT
    );
