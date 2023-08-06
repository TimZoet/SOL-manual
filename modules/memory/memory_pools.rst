Memory Pools
============

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    pools/free
    pools/non_linear
    pools/ring
    pools/stack

The :code:`Vulkan Memory Allocator` library comes with various memory pools. These are supported by :code:`SOL` as well.
The :code:`sol::IMemoryPool` class defines a generic interface to these pools.

Memory pools can be created through, and are managed by, the :code:`sol::MemoryManager`:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;

    // Create a pool dedicated to vertex and index buffers. Each memory block
    // will be 64MiB, so buffers allocated from this pool cannot be larger.
    // 4 blocks are preallocated and up to 32 blocks can be allocated at once.
    // This will limit total memory usage to 2GiB.
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_TRANSFER_DST_BIT |
                                VK_BUFFER_USAGE_VERTEX_BUFFER_BIT |
                                VK_BUFFER_USAGE_INDEX_BUFFER_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .blockSize            = 1024ull * 1024ull * 64ull,
        .minBlocks            = 4,
        .maxBlocks            = 32
    };
    auto& pool = manager.createFreeAtOnceMemoryPool("geometry", info);

From a pool, you can only allocate buffers with the usage and memory properties that were specified on creation.
Therefore, the only parameter that is required when allocating a new buffer from a pool is its size:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;

    // Create a preallocated 128MiB ring buffer for transfers from CPU > GPU.
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_HOST,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = VMA_ALLOCATION_CREATE_MAPPED_BIT |
                                VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT,
        .blockSize            = 1024ull * 1024ull * 128ull,
        .minBlocks            = 1,
        .maxBlocks            = 1
    };
    auto& pool = manager.createRingBufferMemoryPool("transfer_src", info);

    // Use the sol::IMemoryPool interface for allocating a 1 MiB buffer.
    sol::MemoryPoolBufferPtr buffer = pool.allocateBuffer(1024ull * 1024ull);

In case you also wish to specify only a subset of buffer usage flags (for example if you have a pool for vertex and
index buffers, and want to create a vertex-only buffer), or alignment, the :code:`sol::IMemoryPool::AllocationInfo` can
be used:

.. code-block:: cpp

    constexpr sol::IMemoryPool::AllocationInfo alloc{
        .size        = 1024ull * 1024ull,
        .bufferUsage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT,
        .alignment   = 64ull
    };
    sol::MemoryPoolBufferPtr buffer = pool.allocateBuffer(1024ull * 1024ull);

Because :code:`sol::IMemoryPool` implements the :code:`sol::IBufferAllocator` interface, you can also use the generic
allocation interface. Note that because memory pools only support allocations with very specific settings, you must
ensure that the provided allocation parameters are compatible with the pool:

.. code-block:: cpp

    // Create a 1GiB memory pool for GPU only storage buffers.
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .blockSize            = 1024ull * 1024ull * 1024ull,
        .minBlocks            = 0,
        .maxBlocks            = 1
    };
    auto& pool = manager.createNonLinearMemoryPool("gpu_side_storage", info);

    // Use the sol::IBufferAllocator interface for allocating a 128MiB buffer.
    // Note how the parameters match the pool parameters above.
    sol::IBufferAllocator::AllocationInfo alloc{
        .size = 1024ull * 1024ull * 128ull,
        .bufferUsage = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT,
        .sharingMode = VK_SHARING_MODE_EXCLUSIVE,
        .memoryUsage = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags = 0
    };
    sol::IBufferPtr buffer0 = pool.allocateBuffer(alloc);

    // Use the sol::IMemoryPool interface for allocating the same buffer.
    sol::MemoryPoolBufferPtr buffer1 = pool.allocateBuffer(1024ull * 1024ull * 128ull);

    // Trying to allocate an unsupported buffer type will throw.
    alloc.bufferUsage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    auto buffer2 = pool.allocateBuffer(alloc);
