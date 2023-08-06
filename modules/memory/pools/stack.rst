Stack Memory Pool
=================

The :code:`sol::StackMemoryPool` class implements the :code:`sol::IMemoryPool` interface. It supports an arbitrary
number of memory blocks. It can be created through the :code:`createStackMemoryPool` method of the 
:code:`sol::MemoryManager`:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .blockSize            = 1024ull * 1024ull * 64ull, // 64MiB memory blocks.
        .minBlocks            = 3,                         // Preallocate all memory blocks.
        .maxBlocks            = 3                          // Allocate 3 memory blocks.
    };
    auto& pool = manager.createStackMemoryPool("stack_storage", info);

This pool functions as a last-in-first-out (LIFO) allocator. Each new buffer is placed after the previous one and all
buffers must be deallocated in reverse order. Deallocating a buffer before other buffers placed after it are
deallocated will result in a runtime error on the next allocation. Note that this requirement is slightly relaxed when a
deallocation 'fixes' a previously wrong order:

.. code-block:: cpp

    auto b0 = pool.allocateBuffer(1024ull * 1024ull * 64ull);
    auto b1 = pool.allocateBuffer(1024ull * 1024ull * 48ull);
    auto b2 = pool.allocateBuffer(1024ull * 1024ull * 32ull);
    auto b3 = pool.allocateBuffer(1024ull * 1024ull * 32ull);
    
    // Resetting the pointer will cause a deallocation.
    // Since this was the last allocation, it is valid to do another allocation immediately after.
    b3.reset();
    pool.allocateBuffer(...);

    // This deallocation is in the wrong order.
    // Allocating will throw an exception.
    b1.reset();
    pool.allocateBuffer(...); // <- Will throw.

    // With this deallocation, we effectively fix the order again.
    b2.reset();
    auto bx = pool.allocateBuffer(1024ull * 1024ull * 32ull); // <- Will work again.

    // Since we deallocate everything here, any allocation after will succeed.
    b0.reset();
    bx.reset();

The buffers allocated above would be laid out in memory like this:

.. figure:: /_static/images/memory/stack_memory_pool.svg
    :alt: Diagram of the Stack Memory Pool.
