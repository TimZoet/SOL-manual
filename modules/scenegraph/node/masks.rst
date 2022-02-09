Traversal Masks
===============

Nodes have two 64-bit masks that can be used to more accurately specify the exact contents or purpose of a node. The
main goal of masks is to allow you to keep a single scenegraph (or at least, not too many) containing diverse data.
Masks can be used during traversal to control which nodes to visit, ignore, or even cut off entire branches of the
scenegraph.

Masks can be set and retrieved using :code:`(set|get)GeneralMask` and :code:`(set|get)TypeMask`. The general mask
should be used for all types of traversal, even if the traverser does not support the specific node type. For example, a
forward traverser should still inspect the general mask of a deferred material node. The type mask should only be used
if the traverser explicitly supports the node type. Handling of masks by traversers is covered in 
:doc:`../../render/common/traverser`.

There exist many use cases. One example could be the rendering of both opaque and transparent geometry. This is
best done in two different passes. However, keeping two separate scenegraphs can be inconvenient. The glass
(transparent) and body (opaque) of a vehicle would ideally share a root node containing a transform. By giving the
relevant nodes the right masks, one traversal pass can be made to ignore transparent nodes while another skips opaque
ones. See below for a rudimentary example:

.. code-block:: cpp

    // Best to create an enum instead of hardcoding magic values.
    enum RenderType
    {
        Opaque      = 1,
        Transparent = 2,
        Particle    = 4
    };
    
    sol::ScenegraphPtr scenegraph = ...;
    auto&                rootNode = scenegraph.getRootNode();

    // Setup a basic hierarchy with two material nodes and a mesh node for each.
    // The right material node references a material with transparency.
    auto&   mtlNodeLeft = rootNode.addChild(std::make_unique<sol::ForwardMaterialNode>(...));
    auto&  meshNodeLeft = mtlNodeLeft.addChild(std::make_unique<sol::MeshNode>(...));
    auto&  mtlNodeRight = rootNode.addChild(std::make_unique<sol::ForwardMaterialNode>(...));
    auto& meshNodeRight = mtlNodeRight.addChild(std::make_unique<sol::MeshNode>(...));

    // Mark one node as opaque and the other as transparent. We mark the material node,
    // because continuing traversal to the mesh node is redundant.
    mtlNodeLeft.setTypeMask(RenderType::Opaque);
    mtlNodeRight.setTypeMask(RenderType::Transparent);

    // Setup traversers to take into account type masks.
    ...

Note that since it is possible to create instances of the base class (i.e. :code:`sol::Node`, with :code:`Type::Empty`),
these can be used to direct traversal with their general mask.
