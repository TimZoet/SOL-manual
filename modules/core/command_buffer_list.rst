Command Buffer List
===================

The :code:`sol::VulkanCommandBufferList` class manages the lifetime of multiple :code:`VkCommandBuffers`. These command
buffers are allocated all at once, so using this class is likely to be more efficient than multiple 
:code:`VulkanCommandBuffer` objects (see :doc:`command_buffer`). Apart from that, it is essentially identical to its 
one buffer counterpart. The only differences are of course that you must specify the number of buffers to allocate, and
a buffer index when beginning and ending commands.

.. code-block:: cpp

    // Assuming a previously created command pool.
    sol::VulkanCommandPool& pool = ...;

    // Allocate 3 command buffers at once.
    sol::VulkanCommandBufferList::Settings settings;
    settings.commandPool = pool;
    settings.size        = 3;
    auto bufferList      = sol::VulkanBufferList::create(settings);

    // Record using command buffer 2.
    bufferList->beginCommand(2);
    ... // Record stuff.
    bufferList->endCommand(2);
