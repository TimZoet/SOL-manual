Scenegraph
==========

The :code:`sol::Scenegraph` class mainly functions as a container for nodes. It always holds a single root node of type
:code:`sol::Node`, i.e. the base node class. This node cannot be removed. Adding, modifying and removing nodes is done
mainly through the node classes:

.. code-block:: cpp
    
    auto     scenegraph = std::make_unique<sol::Scenegraph>();
    sol::Node& rootNode = scenegraph->getRootNode();

    auto& child0 = rootNode.addChild(std::make_unique<SomeNode>(...));
    auto& child1 = child0.addChild(std::make_unique<SomeNode>(...));
    auto& child2 = child1.addChild(std::make_unique<SomeNode>(...));
    auto& child3 = child1.addChild(std::make_unique<SomeNode>(...));

Subgraphs
---------

.. note::

    Not yet implemented.

Visualization
-------------

.. note::

    Not yet implemented.
