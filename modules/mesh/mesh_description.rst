Mesh Description
================

The :code:`sol::MeshDescription` class can be used to create various types of meshes. It holds persistently mapped 
staging buffers for vertex and index data. You can add any number of vertex buffers using the :code:`addVertexBuffer`
method and exactly one index buffer using the :code:`addIndexBuffer` method. Each buffer can then be filled with vertex
and index data using the various :code:`set` methods.

Below a simple example of how to create and fill a description object with a single vertex buffer and an index buffer:

.. code-block:: cpp

    // Repeating the vertex definition from the MeshLayout:

    struct Vertex
    {
        float3 pos;
        float3 color;
        float2 uv;
    };

    ...

    // Assuming the following vectors filled with vertex attributes and indices:

    std::vector<float3> positions;
    std::vector<float3> colors;
    std::vector<float2> uvs;
    std::vector<uint3> indices;

    ...

    // Create a description with both a vertex and index buffer.
    auto desc = meshManager->createMeshDescription();
    desc->addVertexBuffer(sizeof(Vertex), positions.size());
    desc->addIndexBuffer(sizeof(uint32_t), indices.size() * 3);

    // Vertex attributes are in separate vectors and need to be 
    // combined into a single struct and copied per-element.
    for (size_t i = 0; i < positions.size(); i++)
    {
        Vertex v{.pos = positions[i], .color = colors[i], .uv = uvs[i]};
        desc->setVertexData(0, i, &v);
    }

    // Vector with indices contains contiguous data and can be copied all at once.
    desc->setIndexData(0, indices.size() * 3, indices.data());

After the mesh description has been filled, a mesh can be created. Of course the description should be compatible with
the type of mesh you wish to create. It is for example not possible to create an indexed mesh when there is no index 
staging buffer:

.. code-block:: cpp

    sol::IndexedMesh& mesh = meshManager->createIndexedMesh(std::move(desc));
