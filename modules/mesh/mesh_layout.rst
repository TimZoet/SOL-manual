Mesh Layout
===========

The :code:`sol::MeshLayout` class is used to describe the layout of the vertex shader input attributes and bindings.
Once finalized, the descriptions it generates are used to create the vertex shader stage of a graphics pipeline. 
Consider the following vertex shader and corresponding C++ class:

.. code-block:: cpp

    // Vertex shader:

    layout(location = 0) in vec3 inPos;
    layout(location = 1) in vec3 inColor;
    layout(location = 2) in vec2 inUv;

    ...

    // C++ vertex definition:

    struct Vertex
    {
        float3 pos;
        float3 color;
        float2 uv;
    };

    ...

This particular layout can be specified by creating a new :code:`sol::MeshLayout` through the :code:`sol::MeshManager`
and adding the various attributes and bindings:

.. code-block:: cpp

    auto& meshLayout = meshManager.createMeshLayout("basiclayout");
    meshLayout.addAttribute("position", 0, 0, VK_FORMAT_R32G32B32_SFLOAT, offsetof(Vertex, pos));
    meshLayout.addAttribute("color", 1, 0, VK_FORMAT_R32G32B32_SFLOAT, offsetof(Vertex, color));
    meshLayout.addAttribute("uv", 2, 0, VK_FORMAT_R32G32_SFLOAT, offsetof(Vertex, uv));
    meshLayout.addBinding("vertex", 0, sizeof(Vertex), VK_VERTEX_INPUT_RATE_VERTEX);
    meshLayout.finalize();

The layout must be finalized before it can be used during rendering. After finalization, it can no longer be modified.
