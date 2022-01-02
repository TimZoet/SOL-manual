Shared Mesh
===========

All other mesh classes own their vertex and index buffers. As such, it is difficult to have different meshes share data,
despite there being many use cases. For example, you might have one very large vertex buffer and multiple index buffers
that should share the vertex buffer. That is what the :code:`sol::SharedMesh` class is for. It does not hold any
buffers, but instead refers to the buffers from other meshes.

When translated to code, the above example would roughly look like this:

.. code-block:: cpp

    // A mesh with a very large vertex buffer.
    sol::FlatMesh& vertexMesh = meshManager->createFlatMesh(...);

    // A list of meshes with small index buffers.
    std::vector<sol::IndexedMesh*> indexMeshes;
    for (...)
    {
        indexmeshes.emplace_back(&meshManager->createIndexedMesh(...));
    }

    // Create a shared mesh that inherits its buffers from the vertex mesh and one of the index meshes.
    std::vector<sol::SharedMesh*> sharedMeshes;
    for (auto* indexMesh : indexMeshes)
    {
        auto& sharedMesh = meshManager->createSharedMesh();
        sharedMesh.addVertexBuffer(vertexMesh);
        sharedMesh.setIndexBuffer(indexMesh);
    }

By default a :code:`sol::SharedMesh` inherits all its settings (offsets, counts, etc.) from the meshes added to it.
However, you can also override these values. For example, you can define an offset in the index buffer using the 
:code:`setFirstIndex` method. That way, multiple shared meshes can read the same index buffer, only starting at
different locations.

Since the shared mesh does not own the buffers, nor the meshes it inherits from, you must make sure that these are not
destroyed before the shared mesh. This is not handled automatically.
