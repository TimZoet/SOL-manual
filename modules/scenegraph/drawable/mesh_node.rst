Mesh Node
=========

The :code:`sol::MeshNode` holds a single reference to a :code:`sol::MeshInstance`. The mesh is rendered with the 
material that is bound by the nearest ancestor material node.

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::MeshInstance&          mesh0 = ...;
    sol::MeshInstance&          mesh1 = ...;
    sol::ForwardMaterialInstance& mtl = ...;
    sol::Scenegraph        scenegraph = ...;
    auto&                        root = scenegraph.getRootNode();
    auto&                materialNode = root.addChild(std::make_unique<sol::ForwardMaterialNode>(mtl));

    // Both meshes will be rendered with the material from the node created above.
    materialNode.addChild(std::make_unique<sol::MeshNode>(mesh0));
    materialNode.addChild(std::make_unique<sol::MeshNode>(mesh1));
