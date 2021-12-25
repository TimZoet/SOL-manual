Submit Command
==============

The :code:`sol::SubmitCommand` can be used to submit one or more command buffers to a queue. You can make use of a
:code:`sol::VulkanCommandBufferList` or separate :code:`sol::VulkanCommandBuffer` objects:

.. code-block:: cpp

    sol::CommandQueue&                 commandQueue = ...;
    sol::VulkanQueue&                         queue = ...;
    sol::VulkanCommandBufferList& commandBufferList = ...;
    sol::VulkanCommandBuffer&        commandBuffer0 = ...;
    sol::VulkanCommandBuffer&        commandBuffer1 = ...;

    // Submit all command buffers managed by a list:
    auto& submitCommand0 = commandQueue.createCommand<sol::SubmitCommand>();
    submitCommand0.setQueue(queue);
    submitCommand0.setCommandBufferList(commandBufferList);

    // Submit separately managed command buffers:
    auto& submitCommand1 = commandQueue.createCommand<sol::SubmitCommand>();
    submitCommand1.setQueue(queue);
    submitCommand1.addCommandBuffer(commandBuffer0);
    submitCommand1.addCommandBuffer(commandBuffer1);

It is also possible to provide an index parameter so submit only one of the command buffers:

.. code-block:: cpp

    uint32_t frameIndex = 0;
    submitCommand.setCommandBufferIndexPtr(&frameIndex)

Addtionally, wait and signal semaphores can be added. These too support an index parameter to select just one:

.. code-block:: cpp

    sol::VulkanSemaphore&   waitSem0 = ...;
    sol::VulkanSemaphore&   waitSem1 = ...;
    sol::VulkanSemaphore& signalSem0 = ...;
    sol::VulkanSemaphore& signalSem1 = ...;
    uint32_t               waitIndex = 0;

    submitCommand.addWaitSemaphore(waitSem0, VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT);
    submitCommand.addWaitSemaphore(waitSem1, VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT);
    submitCommand.setWaitSemaphoreIndexPtr(&waitIndex);

    submitCommand.addSignalSemaphore(signalSem0);
    submitCommand.addSignalSemaphore(signalSem1);

Finally, one fence (possibly selected using an index) can be signalled:

.. code-block:: cpp

    sol::VulkanFence& signalFence = ...;

    submitCommand.addSignalFence(signalFence);
