Acquire Command
===============

The :code:`sol::AcquireCommand` can be used to acquire the next available image from a swapchain. There are optional
parameters to pass a semaphore and fence name to be signaled. The index of the available image is written to the integer
pointer parameter:

.. code-block:: cpp

    sol::CommandQueue&    commandQueue = ...;
    sol::VulkanSwapchain&    swapchain = ...;
    sol::VulkanSemaphore&    signalSem = ...;
    sol::VulkanFence&      signalFence = ...;
    uint32_t                imageIndex = 0;

    // Acquire image from swapchain and signal both a semaphore and fence.
    auto& acquireCommand = commandQueue.createCommand<sol::AcquireCommand>();
    acquireCommand.setSwapchain(swapchain);
    acquireCommand.addSignalSemaphore(signalSem);
    acquireCommand.addSignalFence(signalFence);
    acquireCommand.setImageIndexPtr(&imageIndex);

It is also possible to add multiple semaphores or fences and signal one of them based on an index parameter. This can be 
used e.g. when rendering to different images each frame:

.. code-block:: cpp

    sol::VulkanSemaphore&    signal0 = ...;
    sol::VulkanSemaphore&    signal1 = ...;
    uint32_t              frameIndex = 0;

    // Acquire image from swapchain and signal one of the semaphores.
    acquireCommand.addSignalSemaphore(signal0);
    acquireCommand.addSignalSemaphore(signal1);
    acquireCommand.setSignalSemaphoreIndexPtr(&frameIndex);
