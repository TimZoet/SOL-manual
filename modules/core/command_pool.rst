Command Pool
============

The :code:`sol::VulkanCommandPool` class manages the lifetime of a :code:`VkCommandPool`. With a command pool you can
allocate :code:`sol::VulkanCommandBuffer` and :code:`sol::VulkanCommandBufferList` objects (see also 
:doc:`command_buffer` and :doc:`command_buffer_list`).

.. code-block:: cpp

    // Creating a graphics command pool with resettable buffers.
    VulkanCommandPool::Settings settings;
    settings.device           = device;
    settings.queueFamilyIndex = physicalDevice.getQueueFamilyIndices().getGraphicsFamily();
    settings.flags            = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
    commandPool               = VulkanCommandPool::create(settings);
