Queue Family
============

The :code:`sol::VulkanQueueFamily` class is a utility that keeps track of the supported features of a queue family. The
:code:`sol::VulkanPhysicalDevice` keeps a list of instances of this class, one for each queue family that is supported
on the physical device. This can be used to e.g. find an actual :code:`VulkanQueue` instance to perform certain
operations:

.. code-block:: cpp

    sol::VulkanQueueFamily& getBestTransferQueueFamily(sol::VulkanPhysicalDevice& device)
    {
        // Look for dedicated transfer queue.
        for (auto& family : device.getQueueFamilies())
        {
            if (family.supportsDedicatedTransfer()) return family;
        }

        // No dedicated transfer queue, guess we'll have to use a non-dedicated one.
        for (auto& family : device.getQueueFamilies())
        {
            if (family.supportsTransfer()) return family;
        }

        // Technically shouldn't be possible to get here...
        throw std::runtime_error("No transfer queue family found.");
    }
