Forward Material Node
=====================

The :code:`sol::ForwardMaterialNode` holds a single reference to a :code:`sol::ForwardMaterialInstance`. Any objects 
below it in the hierarchy are rendered using the assigned material instance, until another material node is encountered
that overrides the material again.

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::MeshInstance&            mesh = ...;
    sol::ForwardMaterialInstance& mtl0 = ...;
    sol::ForwardMaterialInstance& mtl1 = ...;
    sol::Scenegraph         scenegraph = ...;
    auto&                         root = scenegraph.getRootNode();

    // The first mesh is rendered using mtl0.
    auto& mtlNode0  = root.createChild<sol::ForwardMaterialNode>(mtl0);
    auto& meshNode0 = mtlNode0.createChild<sol::MeshNode>(mesh);

    // The mesh below it with mtl1.
    auto& mtlNode1  = meshNode0.createChild<sol::ForwardMaterialNode>(mtl1);
    auto& meshNode1 = mtlNode1.createChild<sol::MeshNode>(mesh);
