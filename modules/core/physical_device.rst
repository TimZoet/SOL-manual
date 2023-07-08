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

Querying for features requires setting up potentially long chains of feature structs. To make this slightly less
painful, the :code:`sol::VulkanPhysicalDeviceFeatures2` struct can be used to automate most of this process. It takes a
list of feature struct type parameters. For example, to query for a device with ray tracing features, we'd need the
following chain:

.. code-block:: cpp

    auto supportedFeatures = std::make_unique<sol::VulkanPhysicalDeviceFeatures2<
        sol::VulkanPhysicalDeviceVulkan11Features,
        sol::VulkanPhysicalDeviceVulkan12Features,
        sol::VulkanPhysicalDeviceVulkan13Features,
        sol::VulkanPhysicalDeviceAccelerationStructureFeaturesKHR,
        sol::VulkanPhysicalDeviceRayQueryFeaturesKHR,
        sol::VulkanPhysicalDeviceRayTracingPipelineFeaturesKHR
    >>();

These features are filled during device selection and passed to a potential feature filter function:

.. code-block:: cpp

    sol::VulkanPhysicalDevice::Settings settings;
    settings.features      = supportedFeatures.get();
    settings.featureFilter = [](const sol::RootVulkanPhysicalDeviceFeatures2& features) {
        // Note how you don't have to traverse the chain yourself,
        // but can just retrieve the structures directly.
        if (!features.getAs<sol::VulkanPhysicalDeviceVulkan12Features>()->bufferDeviceAddress) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceVulkan12Features>()->descriptorIndexing) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceVulkan13Features>()->dynamicRendering) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceVulkan13Features>()->synchronization2) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceAccelerationStructureFeaturesKHR>()->accelerationStructure) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceRayQueryFeaturesKHR>()->rayQuery) return false;
        if (!features.getAs<sol::VulkanPhysicalDeviceRayTracingPipelineFeaturesKHR>()->rayTracingPipeline) return false;

        return true;
    };

    // During device selection, vkGetPhysicalDeviceFeatures2 is called.
    // Results are written to the features pointer, which is then passed
    // to the filter function.
    auto physicalDevice = sol::VulkanPhysicalDevice::create(settings);

Vulkan has a large number of feature structs, not all of which have been exposed by :code:`SOL` yet. If you need a
struct that is missing, just create a new :code:`using` definition, passing the :code:`Vk...Features` struct and the
structure type enum value to :code:`sol::VulkanPhysicalDeviceFeature`. For example,
:code:`VkPhysicalDeviceRayTracingPipelineFeaturesKHR` is defined as follows:

.. code-block:: cpp

    using VulkanPhysicalDeviceRayTracingPipelineFeaturesKHR =
        sol::VulkanPhysicalDeviceFeature<
            VkPhysicalDeviceRayTracingPipelineFeaturesKHR,
            VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_RAY_TRACING_PIPELINE_FEATURES_KHR
        >;

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
