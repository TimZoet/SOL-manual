Mesh Collection
===============

The :code:`sol::MeshCollection` class manages a collection of :doc:`/modules/mesh/meshes`. It has a simple interface
for creating and destroying these objects. It does not take care of allocating vertex and index buffers itself. For that
it relies on the :doc:`/modules/mesh/geometry_buffer_allocator`.

.. code-block:: cpp
    :caption: Creating a mesh collection.

    sol::GeometryBufferAllocator&  allocator = ...;
    sol::MeshCollectionPtr        collection = std::make_unique<sol::MeshCollection>(allocator);



You can create any number of mesh collections, each managing their own set of resources. This can be useful to keep
different parts of your application completely independent, and simplify the lifetime management of sets of related
resources. Also, a single mesh collection is not thread safe. Using multiple collections can help avoid expensive
locking mechanisms.

To simplify allocating buffers, you can do so directly through the collection.

.. code-block:: cpp
    :caption: Allocating buffers through the collection.

    sol::MeshCollection& collection = ...;

    // No need to get allocator...
    auto buffer = collection.getAllocator().allocateVertexBuffer(1024, sizeof(Vertex));
    // ...you can just do this, which reroutes the call.
    auto buffer = collection.allocateVertexBuffer(1024, sizeof(Vertex));
