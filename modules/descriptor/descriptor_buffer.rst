Descriptor Buffer
=================

The :code:`sol::DescriptorBuffer` class holds a single :code:`IBuffer` from which descriptors can be allocated. Creating
it requires only a few settings:

.. code-block:: cpp
    :caption: Creating a 128MiB descriptor buffer.

    sol::Memorymanager& manager = ...;

    const sol::DescriptorBuffer::Settings settings{
        .memoryManager = &getMemoryManager(),
        // 128MiB.
        .size          = 128ULL * 1024 * 1024,
        // Using it for resource descriptors and transfers to and from.
        .usageFlags    = VK_BUFFER_USAGE_RESOURCE_DESCRIPTOR_BUFFER_BIT_EXT |
                         VK_BUFFER_USAGE_TRANSFER_DST_BIT |
                         VK_BUFFER_USAGE_TRANSFER_SRC_BIT
    };

    auto buffer = sol::DescriptorBuffer::create(settings);

The default value for :code:`usageFlags` is :code:`VK_BUFFER_USAGE_RESOURCE_DESCRIPTOR_BUFFER_BIT_EXT | VK_BUFFER_USAGE_SAMPLER_DESCRIPTOR_BUFFER_BIT_EXT`.
In the example above we are restricting it to just :code:`_RESOURCE_DESCRIPTOR_`. Also, maybe we wish to perform copies
involving this descriptor buffer, so we enable both transfer flags. Internally, the
:code:`VK_BUFFER_USAGE_SHADER_DEVICE_ADDRESS_BIT` flag is always enabled.

To allocate a :doc:`/modules/descriptor/descriptor`, you need a valid :doc:`/modules/descriptor/descriptor_layout`:

.. code-block:: cpp
    :caption: Allocating descriptors from a buffer.

    sol::DescriptorBuffer& buffer = ...;

    // Allocate 10 descriptors with layout0.
    sol::DescriptorLayout& layout0 = ...;
    std::vector<sol::DescriptorPtr> descriptors0;
    for (size_t i = 0; i < 10; i++>)
        descriptors0.emplace_back(buffer.allocateDescriptor(layout0));
    
    // Allocate 100 descriptors with layout1.
    sol::DescriptorLayout& layout1 = ...;
    std::vector<sol::DescriptorPtr> descriptors1;
    for (size_t i = 0; i < 100; i++>)
        descriptors1.emplace_back(buffer.allocateDescriptor(layout1));

A descriptor is assigned a region in the buffer that it holds until it goes out of scope. On creation, this memory
obviously does not contain any descriptors yet. You can fill these through the descriptor object.
