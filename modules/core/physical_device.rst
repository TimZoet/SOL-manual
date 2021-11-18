Physical Device
===============

The :code:`sol::VulkanPhysicalDevice` class manages the lifetime of a :code:`VkPhysicalDevice`. There might be multiple 
physical devices available on a machine. Furtermore, you most likely need a device that has support for certain features
and extensions. This can all be specified:

.. code-block:: cpp

    sol::VulkanPhysicalDevice::Settings settings;
    settings.instance = instance;

    // Setting the surface is optional.
    settings.surface = surface;
    
    // List of device extensions to be enabled. 
    settings.extensions.emplace_back(VK_KHR_SWAPCHAIN_EXTENSION_NAME);

    // We only want to consider discrete GPUs. Integrated GPUs are too slow.
    settings.propertyFilter = [](const VkPhysicalDeviceProperties& props) {
        return props.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU;
    };

    // We want a device that supports geometry and tessellation shaders, because why not.
    settings.featureFilter = [](const VkPhysicalDeviceFeatures& features) {
        return features.geometryShader && features.tessellationShader;
    };

    // We require graphics and present queues, and also want to have a dedicated transfer queue.
    settings.queueFamilyFilter = [](const std::vector<sol::VulkanQueueFamily>& queues) {
        bool hasGraphics = false;
        bool hasPresent = false;
        bool hasDedicatedTransfer = false;

        for (const auto& q : queues)
        {
            if (q.supportsGraphics())
                hasGraphics = true;
            if (q.supportsPresent())
                hasPresent = true;
            if (q.supportsTransfer() && !q.supportsCompute() && !q.supportsGraphics() && !q.supportsPresent())
                hasDedicatedTransfer = true;
        }

        return hasGraphics && hasPresent && hasDedicatedTransfer;
    };

    auto physicalDevice = sol::VulkanPhysicalDevice::create(settings);

Surface and Swapchain
---------------------

.. note ::

    Not yet implemented.
