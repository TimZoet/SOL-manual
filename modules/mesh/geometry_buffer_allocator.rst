Geometry Buffer Allocator
=========================

The :code:`sol::GeometryBufferAllocator` class provides a simple interface for allocating
:doc:`/modules/mesh/vertex_buffers` and :doc:`/modules/mesh/index_buffers`. It can operate in two different modes. The
first is by using the :code:`Separate` strategy. Each allocation will create a wholly separate buffer.

.. code-block:: cpp
    :caption: Creating an allocator that will allocate separate buffers.

    const sol::GeometryBufferAllocator::Settings settings{
        .memoryManager = getMemoryManager(),
        .strategy = sol::GeometryBufferAllocator::Strategy::Separate
    };

    const auto allocator = sol::GeometryBufferAllocator::create(settings);

    // Allocate two buffers. Note how vertex sizes are different.
    sol::VertexBufferPtr b0 = allocator->allocateVertexBuffer(1024, 16);
    sol::VertexBufferPtr b1 = allocator->allocateVertexBuffer(2048, 32);

    // Neither buffers are suballocations...
    assert(!b0->isSubAllocation() && !b1->isSubAllocation());
    // ...meaning their buffers must be different objects altogether.
    assert(&b0->getBuffer() != &b1->getBuffer());

Because allocating many small buffers can be inefficient, the allocator can also operate using the :code:`Global`
strategy. It will then preallocate a fixed vertex and index buffer from which you can create suballocations.

.. code-block:: cpp
    :caption: Creating an allocator that will allocate two global buffers.

    const sol::GeometryBufferAllocator::Settings settings{
        .memoryManager = getMemoryManager(),
        .strategy = sol::GeometryBufferAllocator::Strategy::Global,
        .vertexCount = 1024 * 1024,
        .vertexSize  = 16,
        .indexCount  = 1024 * 1024,
        .indexSize   = 4
    };
    
    // This allocator will manage a 16MiB vertex buffer and 4MiB index buffer.
    const auto allocator = sol::GeometryBufferAllocator::create(settings);

    // We can no longer specify a vertex (or index) size. Any value will get ignored.
    sol::VertexBufferPtr b0 = allocator->allocateVertexBuffer(1024); // size = 16
    sol::VertexBufferPtr b1 = allocator->allocateVertexBuffer(2048); // size = 16

    // These buffers are suballocations...
    assert(b0->isSubAllocation() && b1->isSubAllocation());
    // ...from the same buffer managed by the allocator.
    assert(&b0->getBuffer() == &b1->getBuffer());

If you need to allocate meshes with varying vertex and index sizes, you can of course create more than one allocator.

Regardless of chosen strategy, an allocator should not be destroyed before any buffers allocated from it.