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
    settings.featureFilter = [](const VkPhysicalDeviceFeatures2& features2) {
        return features2.features.geometryShader && features2.features.tessellationShader;
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

Extensions
----------

.. note::

    TODO: More extensive explanation for enabling extensions.

Features
--------

Depending on your needs, it is possible to query for and filter based on features in different ways. Firstly, with just
a simple feature filter:

.. code-block:: cpp

    // Specify a filter, but provide no pointer to a features2 struct.
    sol::VulkanPhysicalDevice::Settings settings;
    settings.featureFilter = [](const VkPhysicalDeviceFeatures2& features2) {
        return features2.features.geometryShader && features2.features.tessellationShader;
    };

    // During device selection, vkGetPhysicalDeviceFeatures2 is called
    // with a temporary features2 struct that is passed to the filter.
    // The pNext chain is not filled with anything, so we can only check
    // for core features. Also, we cannot access the features2 struct
    // after device creation and will have to explicitly call
    // vkGetPhysicalDeviceFeatures2 again.
    auto physicalDevice = sol::VulkanPhysicalDevice::create(settings);

Additionally, you can keep the features struct around:

.. code-block:: cpp

    VkPhysicalDeviceFeatures2 features2{};

    // Specify a features2 struct but no explicit filter.
    sol::VulkanPhysicalDevice::Settings settings;
    settings.features = &features2;

    // During device selection, vkGetPhysicalDeviceFeatures2 is called
    // with the specified features2 struct. No filtering is done,
    // we are happy with any device we get.
    auto physicalDevice = sol::VulkanPhysicalDevice::create(settings);

    // You can still inspect the supported features afterwards.
    if (features2.features.geometryShader) ...

To do a much more extensive query for the supported features, for example if you wish to enable ray tracing, you can
set up the :code:`pnext` chain as follows:

.. code-block:: cpp

    struct MyFeatures
    {
        VkPhysicalDeviceFeatures2 features2{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2 };

        VkPhysicalDeviceVulkan11Features features11{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_1_FEATURES };
        VkPhysicalDeviceVulkan12Features features12{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_2_FEATURES };
        VkPhysicalDeviceVulkan13Features features13{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_3_FEATURES };

        VkPhysicalDeviceAccelerationStructureFeaturesKHR accelerationStructure{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_ACCELERATION_STRUCTURE_FEATURES_KHR };
        VkPhysicalDeviceRayQueryFeaturesKHR rayQuery{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_QUERY_FEATURES_KHR };
        VkPhysicalDeviceRayTracingPipelineFeaturesKHR rayTracingPipeline{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_TRACING_PIPELINE_FEATURES_KHR };
    };

    // Setup feature chain.
    MyFeatures supportedFeatures;
    supportedFeatures.features2.pNext             = &supportedFeatures.features11;
    supportedFeatures.features11.pNext            = &supportedFeatures.features12;
    supportedFeatures.features12.pNext            = &supportedFeatures.features13;
    supportedFeatures.features13.pNext            = &supportedFeatures.accelerationStructure;
    supportedFeatures.accelerationStructure.pNext = &supportedFeatures.rayQuery;
    supportedFeatures.rayQuery.pNext              = &supportedFeatures.rayTracingPipeline;

    // Specify both struct and filter.
    sol::VulkanPhysicalDevice::Settings settings;
    settings...             = ...;
    settings.features       = &supportedFeatures.features2;
    settings.featureFilter  = [](const VkPhysicalDeviceFeatures2& f) {
        // Traverse the chain of supported features, ensuring that everything we need for ray tracing is available.
        const auto* next = reinterpret_cast<const VkBaseOutStructure*>(&f);
        while (next)
        {
            if (next->sType == VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_2_FEATURES)
            {
                const auto* s = reinterpret_cast<const VkPhysicalDeviceVulkan12Features*>(next);
                if (!s->bufferDeviceAddress) return false;
                if (!s->descriptorIndexing) return false;
            }
            else if (next->sType == VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_3_FEATURES)
            {
                const auto* s = reinterpret_cast<const VkPhysicalDeviceVulkan13Features*>(next);
                if (!s->synchronization2) return false;
            }
            else if (next->sType == VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_ACCELERATION_STRUCTURE_FEATURES_KHR)
            {
                const auto* s = reinterpret_cast<const VkPhysicalDeviceAccelerationStructureFeaturesKHR*>(next);
                if (!s->accelerationStructure) return false;
            }
            else if (next->sType == VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_QUERY_FEATURES_KHR)
            {
                const auto* s = reinterpret_cast<const VkPhysicalDeviceRayQueryFeaturesKHR*>(next);
                if (!s->rayQuery) return false;
            }
            else if (next->sType == VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_TRACING_PIPELINE_FEATURES_KHR)
            {
                const auto* s = reinterpret_cast<const VkPhysicalDeviceRayTracingPipelineFeaturesKHR*>(next);
                if (!s->rayTracingPipeline) return false;
            }
            next = reinterpret_cast<const VkBaseOutStructure*>(next->pNext);
        }

        return true;
    };
    auto physicalDevice = sol::VulkanPhysicalDevice::create(settings);

Properties
----------

.. note::

    TODO: More extensive explanation for property filtering.

Queues
------

.. note::

    TODO: More extensive explanation for requiring queue families.

Surface and Swapchain
---------------------

.. note ::

    Not yet implemented.
