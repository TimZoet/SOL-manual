Forward Material Node
=====================

The :code:`sol::ForwardMaterialNode` holds a single reference to a :code:`sol::ForwardMaterialInstance`. Any drawables 
below it in the hierarchy are rendered using the assigned material instance, until another material node is encountered
that overrides the material again.

All drawable nodes will be drawn with the material of the nearest material instance node above them and, for each
descriptor set in the material layout, the nearest material instance node representing that set.

.. code-block:: cpp

    // Assuming some previously created objects.
    // Both material instances represent a different material with just one descriptor set.
    sol::MeshInstance&            mesh = ...;
    sol::ForwardMaterialInstance& mtl0 = ...;
    sol::ForwardMaterialInstance& mtl1 = ...;
    sol::Scenegraph         scenegraph = ...;
    auto&                         root = scenegraph.getRootNode();

    // The first mesh is rendered using mtl0.
    auto&  mtlNode0 = root.addChild(std::make_unique<sol::ForwardMaterialNode>(mtl0));
    auto& meshNode0 = mtlNode0.addChild(std::make_unique<sol::MeshNode>(mesh));

    // The mesh below it using mtl1.
    auto&  mtlNode1 = meshNode0.addChild(std::make_unique<sol::ForwardMaterialNode>(mtl1));
    auto& meshNode1 = mtlNode1.addChild(std::make_unique<sol::MeshNode>(mesh));

.. note::

    There isn't yet a node that can hold multiple material instances (representing multiple descriptor sets of the same
    material).
