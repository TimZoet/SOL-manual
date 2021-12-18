Descriptor Pool
===============

The :code:`sol::VulkanDescriptorPool` class manages the lifetime of a :code:`VkDescriptorPool`. Creation requires the
total number of descriptor sets that can be allocated from the pool, as well as a list of the number of descriptor sets
per descriptor type that should be available:

.. code-block:: cpp

    sol::VulkanDescriptorPool::Settings settings;
    settings.device      = device
    settings.maxSets     = 32;
    settings.poolSizes.emplace_back(VkDescriptorPoolSize{
        .type            = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER,
        .descriptorCount = 32
    });
    settings.poolSizes.emplace_back(VkDescriptorPoolSize{
        .type            = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER,
        .descriptorCount = 16
    });

    auto descSet = sol::VulkanDescriptorPool::create(settings);


.. note::
    There is no functionality yet to track allocations of descriptor sets from a pool.
