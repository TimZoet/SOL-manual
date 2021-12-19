Memory Allocator
================

The :code:`sol::VulkanMemoryAllocator` class manages the lifetime of a :code:`VmaAllocator`. Creation only requires a 
:code:`sol::VulkanDevice`:

.. code-block:: cpp

    sol::VulkanDevice& device = ...;

    sol::VulkanMemoryAllocator::Settings settings;
    settings.device = device;
    auto allocator  = sol::VulkanMemoryAllocator::create(settings);
