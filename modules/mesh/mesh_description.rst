Mesh Description
================

The :code:`sol::MeshDescription` class has but one purpose: transferring mesh data to the GPU. It currently is very 
limited in features. It holds one or two persistently mapped, CPU only buffers: one for vertex data and, optionally, 
one for index data.

On construction, all buffers and settings are determined. These are fixed during the lifetime of the object. Below a 
simple example of how to create and fill a description object:

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

    // Create a description with both vertex and index buffers.
    auto desc = std::make_unique<MeshDescription>(
        meshManager,
        sizeof(Vertex),
        positions.size(),
        sizeof(uint32_t),
        indices.size() * 3
    );

    // Vertex attributes are in separate vectors and need to be 
    // combined into a single struct and copied per-element.
    for (size_t i = 0; i < positions.size(); i++)
    {
        Vertex v{.pos = positions[i], .color = colors[i], .uv = uvs[i]};
        desc->setVertexData(i, &v);
    }

    // Vector with indices contains contiguous data and can be copied all at once.
    desc->setIndexData(0, indices.size() * 3, indices.data());

Once the :code:`sol::MeshDescription` is filled, it can be passed to the :code:`sol::MeshManager` to create a 
:code:`sol::MeshInstance`.
