Free At Once Memory Pool
========================

The :code:`sol::FreeAtOnceMemoryPool` class implements the :code:`sol::IMemoryPool` interface. It supports an arbitrary
number of memory blocks. It can be created through the :code:`createFreeAtOnceMemoryPool` method of the 
:code:`sol::MemoryManager`:

.. code-block:: cpp

    sol::MemoryManager& manager = ...;
    constexpr sol::IMemoryPool::CreateInfo info{
        .createFlags          = 0,
        .bufferUsage          = VK_BUFFER_USAGE_TRANSFER_DST_BIT |
                                VK_BUFFER_USAGE_VERTEX_BUFFER_BIT |
                                VK_BUFFER_USAGE_INDEX_BUFFER_BIT,
        .memoryUsage          = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE,
        .requiredMemoryFlags  = 0,
        .preferredMemoryFlags = 0,
        .allocationFlags      = 0,
        .blockSize            = 1024ull * 1024ull * 16ull, // 16MiB memory blocks.
        .minBlocks            = 1,                         // Preallocate 1 memory block.
        .maxBlocks            = 2                          // Allocate up to 2 memory blocks.
    };
    auto& pool = manager.createFreeAtOnceMemoryPool("geometry", info);

The pool effectively functions as a stack, with each new allocation being placed after the previous one. As its name
implies, all allocated buffers must be deallocated at the same time. The moment one buffer is deallocated, all other
buffers need to be destroyed before a new allocation can take place. Allocating a new buffer while only part of the
previously allocated buffers were released will result in a runtime error:

.. code-block:: cpp

    auto b0 = pool.allocateBuffer(1024ull * 1024ull * 6ull);
    auto b1 = pool.allocateBuffer(1024ull * 1024ull * 9ull);
    auto b2 = pool.allocateBuffer(1024ull * 1024ull * 4ull);
    auto b3 = pool.allocateBuffer(1024ull * 1024ull * 4ull);
    auto b4 = pool.allocateBuffer(1024ull * 1024ull * 4ull);
    
    // Resetting the pointer will cause a deallocation.
    // While the exact order of deallocations does not matter,
    // all buffers must be deallocated before a new allocation takes place.
    b0.reset();
    b4.reset();
    b1.reset();

    // Though there is enough free space, this allocation would result in an exception
    // because not all buffers were released:
    auto b = pool.allocateBuffer(1024ull * 1024ull * 8ull);

    b2.reset();
    b3.reset();

    // All buffers were deallocated, so we can allocate again.
    auto b = pool.allocateBuffer(1024ull * 1024ull * 8ull);

The buffers allocated above would be laid out in memory like this:

.. figure:: /_static/images/memory/free_at_once_memory_pool.svg
    :alt: Diagram of the Free At Once Memory Pool.
