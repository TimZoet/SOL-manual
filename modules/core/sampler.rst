Sampler
=======

The :code:`sol::VulkanSampler` class manages the lifetime of a :code:`VkSampler`. It has a whole bunch of settings:

.. code-block:: cpp

    sol::VulkanDevice& device = ...;

    sol::VulkanSampler::Settings settings;
    settings.device       = device;
    settings.magFilter    = VK_FILTER_LINEAR;
    settings.minFilter    = VK_FILTER_LINEAR;
    settings.addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    settings.addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    settings.addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    ...
    auto sampler          = sol::VulkanSampler::create(setting);
