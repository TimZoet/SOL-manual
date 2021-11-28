Semaphore
=========

The :code:`sol::VulkanSemaphore` class manages the lifetime of a single :code:`VkSemaphore`. They don't require any 
parameters to create, other than a device:

.. code-block:: cpp

    sol::VulkanSemaphore::Settings settings;
    settings.device = device;
    auto semaphore  = sol::VulkanSemaphore::create(settings);

For convenience, the :code:`create` method also allows specifying the number of semaphores to create at once with the
same settings:

.. code-block:: cpp

    // Create 3 semaphores in one go.
    std::vector<sol::VulkanSemaphorePtr> semaphores = sol::VulkanSemaphore::create(settings, 3);
