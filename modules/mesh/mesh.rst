Mesh
====

The :code:`sol::Mesh` class holds a set of one or more vertex buffers and an optional index buffer.

.. code-block:: cpp
    :caption: Creating a mesh with a single vertex and index buffer.

    sol::GeometryBufferAllocator& allocator = ...;

    // We have some data that we want to put into a mesh.
    const std::vector<Vertex>     vertexData = ...;
    sol::VertexBufferPtr        vertexBuffer = allocator.allocateVertexBuffer(vertexData.size(), sizeof(Vertex));
    const std::vector<uint16_t>    indexData = ...;
    sol::IndexBufferPtr          indexBuffer = allocator.allocateIndexBuffer(indexData.size(), sizeof(uint16_t));

    // We can upload the data to the buffers here...
    vertexBuffer->setVertexData(...);
    indexBuffer->setIndexData(...);
    ...

    // Create mesh.
    auto mesh = std::make_unique<sol::Mesh>(std::move(vertexBuffer), std::move(indexBuffer));
    assert(mesh->getVertexBufferCount() == 1);
    assert(mesh->hasIndexBuffer());

    // ...but also here by getting the buffers from the mesh again.
    mesh->getVertexBuffers()[0]->setVertexData(...);
    mesh->getIndexBuffer()->setIndexData(...);
    ...

    // Do some stuff with mesh, like drawing.
    ...

To create meshes with or without index buffers and with many number of vertex buffers, you can do the following:

.. code-block:: cpp
    :caption: Possible configurations of buffers.

    // vertex buffer.
    auto mesh = std::make_unique<sol::Mesh>(std::move(v0));
    // vertex buffer + index buffer.
    auto mesh = std::make_unique<sol::Mesh>(std::move(v0), std::move(i));
    // >1 vertex buffers.
    auto mesh = std::make_unique<sol::Mesh>(std::move(v0), std::move(v1), ...);
    // >1 vertex buffers + index buffer.
    auto mesh = std::make_unique<sol::Mesh>(std::move(v0), std::move(v1), ..., std::move(i));
