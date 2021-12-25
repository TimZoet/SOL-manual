Forward Traverser
=================

The :code:`sol::ForwardTraverser` class transforms the nodes in a :code:`sol::Scenegraph` into a structure that can be
passed on to a :code:`sol::ForwardRenderer`. It will ignore any nodes it does not support.

.. code-block:: cpp

    // Setup scenegraph.
    auto scenegraph = std::make_unique<sol::Scenegraph>();
    ...

    // Setup traverser.
    auto traverser = std::make_unique<sol::ForwardTraverser>();
    ...

    while (rendering)
    {
        // Do other stuff as part of your render loop.
        ...

        // Clear data from previous frame and traverse again.
        traverser->clear();
        traverser->traverse(*scenegraph);

        // Pass traverser to your renderer.
        ...
    }

Configuration
-------------

.. note::
    Not yet implemented.
