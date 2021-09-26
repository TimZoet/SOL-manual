Acquire Command
===============

The :code:`sol::AcquireCommand` can be used to acquire the next available image from a swapchain. There are optional
parameters to pass a semaphore and fence name to be signaled. The index of the available image is written to the integer
pointer parameter:

.. code-block:: cpp

    // Create an acquire command to get the next image and signal a semaphore.
    uint32_t nextImage = 0;
    auto acquireCmd = std::make_unique<sol::AcquireCommand>("mySwapchain", "ImageAvailable", "", &nextImage);

It is also possible to pass a function for generating the semaphore and/or fence that need to be signaled. This can be 
used e.g. when rendering to different images each frame. These functions are invoked whenever the command is executed:

.. code-block:: cpp

    // Create an acquire command to get the next image and signal a dynamic semaphore.
    uint32_t nextImage = 0;
    uint32_t frameIndex = 0;
    auto func = [&frameIndex]() { return std::format("ImageAvailable{0}", frameIndex); };
    auto acquireCmd = std::make_unique<sol::AcquireCommand>("mySwapchain", func, "", &nextImage);
