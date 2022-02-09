ITraverser
==========

The :code:`sol::ITraverser` class defines the interface traverser classes should implement. While this is technically
not required, all built-in traversers do so.

Node Types
----------

.. note::
    Not yet implemented.

Traversal Masks
---------------

As described in :doc:`../../scenegraph/node/masks`, nodes have two masks that can be used to guide traversal. The
:code:`sol::ITraverser` holds two functions that take a mask and return the required action to take. For example,
terminating traversal of a subgraph entirely, or skipping specific nodes.

Continuing the example from the page linked above, we could configure two traversers that render opaque and transparent
objects, respectively, as follows:

.. code-block:: cpp

    sol::ITraverser&      opaqueTraverser = ...;
    sol::ITraverser& transparentTraverser = ...;

    opaqueTraverser.setTypeMaskFunction([](const uint64_t mask) {
        // Continue when encountering a node without mask or with an opaque mask.
        if (mask == 0 || mask & RenderType::Opaque)
            return sol::ITraverser::TraversalAction::Continue;
        // Terminate for all other masks.
        return sol::ITraverser::TraversalAction::Terminate;
    });

    transparentTraverser.setTypeMaskFunction([](const uint64_t mask) {
        // Continue when encountering a node without mask or with a transparent mask.
        if (mask == 0 || mask & RenderType::Transparent)
            return sol::ITraverser::TraversalAction::Continue;
        // Terminate for all other masks.
        return sol::ITraverser::TraversalAction::Terminate;
    });
    