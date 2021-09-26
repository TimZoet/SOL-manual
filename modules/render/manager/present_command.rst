Present Command
===============

The :code:`sol::PresentCommand` can be used to present a swapchain image. To specify the image index, an integer pointer
is needed. Typically, this will be the same pointer as is passed to the :code:`sol::AcquireCommand`. Presentation can 
also wait on a semaphore, for example to display an image only when rendering has finished:

.. code-block:: cpp

    // Probably created an acquire command before:
    uint32_t nextImage = 0;
    auto acquireCmd = std::make_unique<sol::AcquireCommand>("mySwapchain", ..., &nextImage);

    // Create a present command to present the next image. Wait until the semaphore signals rendering is done.
    auto presentCmd = std::make_unique<sol::PresentCommand>("mySwapchain", "RenderFinished", &nextImage);

It is also possible to pass a function for generating the semaphore, e.g. when rendering to different images each frame:

.. code-block:: cpp

    // Create a present command that waits for a dynamic semaphore.
    uint32_t frameIndex = 0;
    auto func = [&frameIndex]() -> std::vector<std::string> { return { std::format("RenderFinished{0}", frameIndex) }; };
    auto presentCmd = std::make_unique<sol::PresentCommand>("mySwapchain", func, &nextImage);
