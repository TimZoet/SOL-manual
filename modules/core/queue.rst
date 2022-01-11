Queue
=====

The :code:`sol::VulkanQueue` class manages a :code:`VkQueue`. The :code:`sol::VulkanDevice` keeps a list of instances of
this class, based on how many were requested during device creation. Because they are a part of a device, queues cannot
be created explicitly. They can only be retrieved from a device:

.. code-block:: cpp

    // Assuming a physical and logical device.
    sol::VulkanDevice& physicalDevice = ...;
    sol::VulkanDevice&         device = ...;

    // Wait for all queues (however many there are) of the compute family to finish executing.
    for (const auto& family : physicalDevice.getQueueFamilies())
    {
        if (!family.supportsCompute()) continue;

        for (auto* queue : device.getQueues(family))
        {
            queue->waitIdle();
        }
    }

Submits
-------

.. note::
    Queue interface needs to be improved a bit before it can be properly documented.

Thread Safety
-------------

By default, queue submits are not thread safe. This is because the underlying Vulkan API function (i.e. 
:code:`vkQueueSubmit`) is not. This can be alleviated by enabling thread safety on the queue using the
:code:`setThreadSafe` method:

.. code-block:: cpp

    sol::VulkanQueue& queue = ...;
    queue.setThreadSafe(true);

    // Any submits beyond this point are thread safe.
    ...

It is also possible to enable thread safety for all queues by default when creating a new :code:`sol::VulkanDevice`:

.. code-block:: cpp

    sol::VulkanDevice::Settings settings;
    settings.threadSafeQueues = true;
    settings...

    auto device = sol::VulkanDevice::create(settings);
