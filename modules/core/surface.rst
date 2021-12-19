Surface
=======

The :code:`sol::VulkanSurface` class manages the lifetime of a :code:`VkSurfaceKHR`. Since this is an extension and you
will typically be using another library for handling surfaces and windows, the creation logic has been slightly 
decoupled. You must specify your own function that will actually create the :code:`VkSurfaceKHR`. If you are e.g. using
the :code:`GLFW` library this could be done as follows:

.. code-block:: cpp

    sol::VulkanSurfacePtr createSurface(sol::VulkanInstance& instance, sol::Window& window)
    {
        sol::VulkanSurface::Settings surfaceSettings;
        surfaceSettings.instance = instance;
        surfaceSettings.func     = [&window](sol::VulkanInstance& instance, VkSurfaceKHR* surface) {
            return glfwCreateWindowSurface(instance.get(), window.get(), nullptr, surface);
        };
        
        return sol::VulkanSurface::create(surfaceSettings);
    }
