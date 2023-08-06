Non Linear Memory Pool
======================

The :code:`sol::NonLinearMemoryPool` class implements the :code:`sol::IMemoryPool` interface. It supports an arbitrary
number of memory blocks. It can be created through the :code:`createNonLinearMemoryPool` method of the 
:code:`sol::MemoryManager`:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .blockSize            = 1024ull * 1024ull * 32ull, // 32MiB memory blocks.
        .minBlocks            = 0,                         // No preallocation.
        .maxBlocks            = 4                          // Allocate up to 4 memory blocks.
    };
    auto& pool = manager.createNonLinearMemoryPool("uniforms", info);

Non linear allocations function just like allocations from global memory. They can be deallocated in random order, with
subsequent allocations trying to fill in any available gaps before allocating new memory blocks.
