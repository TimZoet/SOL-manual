Forward Traverser
=================

The :code:`sol::ForwardTraverser` class transforms the nodes and data in a :code:`sol::Scenegraph` into a
:code:`sol::ForwardRenderData` structure, ready to be passed to a :code:`sol::ForwardRenderer`.

.. code-block:: cpp

    // Assuming some other already initialized objects.
    ...
    auto scenegraph = std::make_unique<sol::Scenegraph>();
    auto renderData = std::make_unique<sol::ForwardRenderData>();
    auto   renderer = std::make_unique<sol::ForwardRenderer>
    ...

    // Setup traverser.
    auto traverser = std::make_unique<sol::ForwardTraverser>();
    ...

    // Render loop.
    while (rendering)
    {
        // Do other stuff.
        ...

        // Clear data from previous frame and traverse again.
        renderData->clear();
        traverser->traverse(*scenegraph, *renderData);

        // Pass data to the renderer.
        ...
    }

Supported Nodes
---------------

.. note::
    Not yet implemented.

Traversal Algorithm
-------------------

.. note::
    Not yet implemented.
