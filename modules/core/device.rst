Device
======

The :code:`sol::VulkanDevice` class manages the lifetime of a :code:`VkDevice`. At the very least, a
:code:`sol::VulkanPhysicalDevice` is needed. Additionally, you'll probably want to specify the same extensions as those
enabled on that physical device. It is also possible to specify how many queues to create for each queue family.
Depending on your needs, you might create queues for just a subset, or many queues for all families:

.. code-block:: cpp

    // Assuming a previously created physical device.
    auto physicalDevice = sol::VulkanPhysicalDevice::create(...);

    sol::VulkanDevice::Settings settings;
    settings.physicalDevice = physicalDevice;

    // Enable all extensions from physical device.
    settings.extensions = physicalDevice.getSettings().extensions;

    // Create a single queue for each queue family.
    settings.queues.resize(physicalDevice.getQueueFamilies().size());
    std::ranges::fill(deviceSettings.queues.begin(), deviceSettings.queues.end(), 1);

    auto device = sol::VulkanDevice::create(settings);
