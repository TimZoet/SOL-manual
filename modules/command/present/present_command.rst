Present Command
===============

The :code:`sol::PresentCommand` can be used to present a swapchain image. To specify the image index, an integer pointer
is needed. Typically, this will be the same pointer as is passed to the :code:`sol::AcquireCommand`. Presentation can 
also wait on one or more semaphores, for example to display an image only when rendering has finished:

.. code-block:: cpp

    sol::CommandQueue&    commandQueue = ...;
    sol::VulkanSwapchain&    swapchain = ...;
    sol::VulkanSemaphore&      waitSem = ...;
    uint32_t                imageIndex = 0;

    // Present image after waiting on semaphore.
    auto& presentCommand = commandQueue.createCommand<sol::PresentCommand>();
    presentCommand.setSwapchain(swapchain);
    presentCommand.addWaitSemaphore(waitSem);
    presentCommand.setImageIndexPtr(&imageIndex);

Presentation can wait on multiple semaphores, or just one of them based on an index parameter:

.. code-block:: cpp

    sol::VulkanSemaphore&      wait0 = ...;
    sol::VulkanSemaphore&      wait1 = ...;
    uint32_t              frameIndex = 0;

    // Present image but wait on one of the semaphores first.
    presentCommand.addWaitSemaphore(wait0);
    presentCommand.addWaitSemaphore(wait1);
    presentCommand.setWaitSemaphoreIndexPtr(&frameIndex);
