Vulkan Error Handler
====================

There is a quite convenient utility function, :code:`sol::handleVulkanError`, which you can use to handle any errors 
returned by Vulkan API function calls. It automatically converts the :code:`VkResult` to a string.
