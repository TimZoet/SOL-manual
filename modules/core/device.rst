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

    // Setup feature chain, matching that of the physical device.
    auto enabledFeatures = std::make_unique<sol::VulkanPhysicalDeviceFeatures2<
        sol::VulkanPhysicalDeviceVulkan11Features,
        sol::VulkanPhysicalDeviceVulkan12Features,
        sol::VulkanPhysicalDeviceVulkan13Features,
        sol::VulkanPhysicalDeviceAccelerationStructureFeaturesKHR,
        sol::VulkanPhysicalDeviceRayQueryFeaturesKHR,
        sol::VulkanPhysicalDeviceRayTracingPipelineFeaturesKHR
    >>();

    // Set all features that must be enabled.
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceVulkan12Features>()->bufferDeviceAddress                   = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceVulkan12Features>()->descriptorIndexing                    = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceVulkan13Features>()->dynamicRendering                      = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceVulkan13Features>()->synchronization2                      = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceAccelerationStructureFeaturesKHR>()->accelerationStructure = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceRayQueryFeaturesKHR>()->rayQuery                           = VK_TRUE;
    enabledFeatures->getAs<sol::VulkanPhysicalDeviceRayTracingPipelineFeaturesKHR>()->rayTracingPipeline       = VK_TRUE;

    sol::VulkanDevice::Settings settings;
    settings...             = ...;
    settings.features       = enabledFeatures.get();
    
    auto device = sol::VulkanDevice::create(settings)

Queues
------

.. note::

    TODO: More extensive explanation for creating queues.
