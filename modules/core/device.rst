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

Extensions
----------

.. note::

    TODO: More extensive explanation for enabling extensions.

Features
--------

Continuing the example of querying for a :doc:`/modules/core/physical_device` with ray tracing support, you can specify
the features that must be enabled when creating a logical device as follows:

.. code-block:: cpp

    // Setup feature chain.
    MyFeatures enabledFeatures;
    enabledFeatures.features2.pNext                             = &enabledFeatures.features11;
    enabledFeatures.features11.pNext                            = &enabledFeatures.features12;
    enabledFeatures.features12.bufferDeviceAddress              = VK_TRUE;
    enabledFeatures.features12.descriptorIndexing               = VK_TRUE;
    enabledFeatures.features12.pNext                            = &enabledFeatures.features13;
    enabledFeatures.features13.synchronization2                 = VK_TRUE;
    enabledFeatures.features13.pNext                            = &enabledFeatures.accelerationStructure;
    enabledFeatures.accelerationStructure.accelerationStructure = VK_TRUE;
    enabledFeatures.accelerationStructure.pNext                 = &enabledFeatures.rayQuery;
    enabledFeatures.rayQuery.rayQuery                           = VK_TRUE;
    enabledFeatures.rayQuery.pNext                              = &enabledFeatures.rayTracingPipeline;
    enabledFeatures.rayTracingPipeline.rayTracingPipeline       = VK_TRUE;
    enabledFeatures.rayTracingPipeline.pNext                    = nullptr;

    // Specify a struct with the enabled features.
    sol::VulkanDevice::Settings settings;
    settings...             = ...;
    settings.features       = &enabledFeatures.features2;
    
    auto device = sol::VulkanDevice::create(settings)

Queues
------

.. note::

    TODO: More extensive explanation for creating queues.
