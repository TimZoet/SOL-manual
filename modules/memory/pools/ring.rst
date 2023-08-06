Ring Buffer Memory Pool
=======================

The :code:`sol::RingBufferMemoryPool` class implements the :code:`sol::IMemoryPool` interface. It supports only one
memory block. It can be created through the :code:`createRingBufferMemoryPool` method of the :code:`sol::MemoryManager`:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_HOST,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT |
                                VMA_ALLOCATION_CREATE_MAPPED_BIT,
        .blockSize            = 1024ull * 1024ull * 128ull, // 128MiB memory blocks.
        .minBlocks            = 1,                          // Preallocate.
        .maxBlocks            = 1                           // Allocate 1 memory block.
    };
    auto& pool = manager.createRingBufferMemoryPool("staging", info);

This pool functions as a first-in-first-out (FIFO) allocator. Each new buffer is placed after the previous one and all
buffers must be deallocated in the same order. Deallocating a buffer before other buffers placed before it are
deallocated may result in a runtime error on the next allocation:

.. code-block:: cpp

    auto b0 = pool.allocateBuffer(1024ull * 1024ull * 32ull);
    auto b1 = pool.allocateBuffer(1024ull * 1024ull * 64ull);
    auto b2 = pool.allocateBuffer(1024ull * 1024ull * 32ull);
    
    // Resetting the pointer will cause a deallocation.
    // Since this was the first allocation,
    // it is valid to do another allocation immediately after.
    b0.reset();
    auto b3 = pool.allocateBuffer(1024ull * 1024ull * 32ull);

    // This deallocation is in the wrong order.
    // Allocating will throw an exception because no empty space is found.
    b2.reset();
    pool.allocateBuffer(1024ull * 1024ull * 32ull); // <- Will throw.

    // With this deallocation, we effectively fix the order again.
    b1.reset();
    pool.allocateBuffer(1024ull * 1024ull * 32ull); // <- Will work again.

The buffers allocated above would be laid out in memory like this:

.. figure:: /_static/images/memory/ring_buffer_memory_pool.svg
    :alt: Diagram of the Ring Buffer Memory Pool.


.. note:: 

    TODO: Describe wait capabilities.
