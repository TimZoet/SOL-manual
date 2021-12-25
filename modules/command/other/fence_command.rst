Fence Command
=============

The :code:`sol::FenceCommand` can be used to wait for and/or reset one or more fences:

.. code-block:: cpp

    sol::CommandQueue& commandQueue = ...;
    sol::VulkanFence&         fence = ...;

    // Wait for a single fence.
    auto& fenceCommand = commandQueue.createCommand<sol::FenceCommand>();
    fenceCommand.setAction(sol::FenceCommand::Action::Wait);
    fenceCommand.addFence(fence);

There are many use cases where a fence command should wait for one of a number of fences. For example, if you have a
render loop where each iteration a different image is used to render to and there is a separate fence for each image. 
For that, it is possible to pass an index parameter:

.. code-block:: cpp

    sol::VulkanFence&     fence0 = ...;
    sol::VulkanFence&     fence1 = ...;
    uint32_t          frameIndex = 0;

    // Wait for and reset one of multiple fences.
    fenceCommand.setAction(sol::FenceCommand::Action::Wait | sol::FenceCommand::Action::Reset);
    fenceCommand.addFence(fence0);
    fenceCommand.addFence(fence1);
    fenceCommand.setFenceIndexPtr(&frameIndex);
