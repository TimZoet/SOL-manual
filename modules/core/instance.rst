Instance
========

The :code:`sol::VulkanInstance` class manages the lifetime of a :code:`VkInstance`. It is of course the starting point
of any Vulkan application. When creating a new instance, you can specify instance extensions, debugging and validation 
layer settings:

.. code-block:: cpp

    sol::VulkanInstance::Settings settings;
    settings.applicationName = "MyRenderer";
    
    // List of instance extensions to be enabled. 
    settings.extensions = sol::Window::getRequiredExtensions();
    
    // When enabling debugging, the VK_EXT_debug_utils extension must be enabled explicitly.
    settings.enableDebugging = true;
    settings.extensions.emplace_back(VK_EXT_DEBUG_UTILS_EXTENSION_NAME);
    
    // Similarly, validation layers can be enabled.
    settings.enableValidation = true;
    settings.validationLayers = {std::string("VK_LAYER_KHRONOS_validation")};

    auto instance = sol::VulkanInstance::create(settings);

