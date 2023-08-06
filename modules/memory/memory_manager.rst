Memory Manager
==============

The :code:`sol::MemoryManager` class is the main interface to memory management. It is used all across :code:`SOL`. Most
higher level modules do not allocate :doc:`/modules/core/buffer` objects directly, but rely on the manager and related
classes instead.

The manager internally uses a VMA :doc:`/modules/core/memory_allocator`. In fact, it cannot function without one. When
constructing the manager, you can either explicitly provide an allocator or have it be constructed for you. Note that
the latter will use all default settings:

.. code-block:: cpp

    // Construct a memory manager with an allocator that has some additional flags enabled.
    sol::VulkanMemoryAllocator::Settings settings;
    settings.device          = *device;
    settings.flags           = VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT;
    auto memoryManager       = std::make_unique<sol::MemoryManager>(sol::VulkanMemoryAllocator::create(settings));

The :code:`sol::MemoryManager` implements the :code:`sol::IBufferAllocator` interface described below. This just
reroutes allocations to the :code:`sol::VulkanBuffer` class, though of course the buffer object is then wrapped in a
:code:`sol::IBuffer`.

IBuffer and IBufferAllocator
----------------------------

There is a generalized interface for buffers and allocators in the :code:`sol::IBuffer` and
:code:`sol::IBufferAllocator` classes.

The allocator interface provides some simple functions for allocating buffers. Implementations are free to allocate
these buffers in any way they see fit, and place restrictions on what types of buffers can be allocated. The order of
allocations and deallocations can also be restricted, for example to create more efficient allocation strategies.
Allocations are done by filling in a :code:`sol::IBufferAllocator::AllocationInfo` struct and passing it to the
:code:`allocateBuffer` method:

.. code-block:: cpp

    // Assuming some allocator implementation.
    sol::IBufferAllocator& allocator = ...;

    // Specify buffer to be allocated.
    constexpr sol::IBufferAllocator::AllocationInfo alloc{
        .size                 = 1024ull,
        .bufferUsage          = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT,
        .sharingMode          = VK_SHARING_MODE_EXCLUSIVE,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .alignment            = 0
    };

    // Allocate buffer.
    sol::IBufferPtr = allocator.allocateBuffer(alloc);

Allocators advertise high-level capabilities through the :code:`sol::IBufferAllocator::Capabilities` enum, which can be
retrieved with the :code:`getCapabilities` method. Trying to use incompatible settings with an allocator can result in
runtime errors.

.. note::

    There is no extensive functionality yet to query the memory manager for allocators that support certain required
    capabilities or types of buffers.

The buffer interface class hides the ownership of underlying :code:`sol::VulkanBuffer` objects. This allows deriving
classes to implement more complex interactions with allocators. For example, several :doc:`/modules/memory/memory_pools`
require deallocating buffers in certain orders, which requires some cooperation from the buffer objects.

.. note::

    TODO: Create an inheritance diagram of allocator and buffer classes included in :code:`SOL`.
