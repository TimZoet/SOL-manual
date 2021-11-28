Fence
=====

The :code:`sol::VulkanFence` class manages the lifetime of a single :code:`VkFence`. Fences can be created in the
unsignaled or signaled state:

.. code-block:: cpp

    // Create unsignaled and already signaled fence.
    sol::VulkanFence::Settings settings;
    settings.device   = device;
    auto fence0       = sol::VulkanFence::create(settings);
    settings.signaled = true;
    auto fence1       = sol::VulkanFence::create(settings);

For convenience, the :code:`create` method also allows specifying the number of fences to create at once with the same
settings:

.. code-block:: cpp

    // Create 3 fences in one go.
    std::vector<sol::VulkanFencePtr> fences = sol::VulkanFence::create(settings, 3);

Creating a fence in the signaled state is useful when you have a fence that synchronizes a loop: 
:code:`wait > do work > signal > wait > do work > signal > ...`. When starting with a signaled fence, you don't need to
have a special case for the first iteration. However, sometimes the work is optional. For example, when dynamically 
loading textures this probably won't need to be done every frame, in which case there is no work that can signal the
fence. You'd probably end up adding a boolean that keeps track of whether there is work to wait on before calling 
:code:`vkWaitForFences`. This boolean is conveniently built into the :code:`sol::VulkanFence` class, although it needs
to be set and reset manually. Calling any Vulkan API function does not affect the value. Also, the get and set are not
thread safe:

.. code-block:: cpp

    // Create a signaled fence.
    sol::VulkanFence::Settings settings;
    settings.device   = device;
    settings.signaled = true;
    auto fence        = sol::VulkanFence::create(settings);
    fence->setSignaled(true);

    ...

    // Use it in a loop somewhere.
    while (someCondition())
    {
        if (fence->isSignaled())
        {
            vkWaitForFences(device->get(), 1, &fence->get(), true, UINT64_MAX);
            vkResetFences(device->get(), 1, &fence->get());
            fence->setSignaled(false);
        }

        if (thereIsWork())
        {
            vkQueueSubmit(..., fence->get());
            fence->setSignaled(true);
        }
    }