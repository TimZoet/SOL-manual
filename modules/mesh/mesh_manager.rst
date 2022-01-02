Mesh Manager
============

The :code:`sol::MeshManager` class manages collections of mesh layouts and meshes. It provides methods to, among other
things, create new layouts and mesh objects. The below diagram shows the relation between objects.

.. figure:: /_static/images/mesh_manager.svg
    :alt: Diagram of the mesh manager class structure.

To construct a new manager a fully initialized :code:`sol::MemoryManager` is needed, as this class provides e.g.
command buffers and queues:

.. code-block:: cpp

    auto memoryManager = std::make_unique<sol::MemoryManager>(...);
    auto meshManager = std::make_unique<sol::MeshManager>(*memoryManager);
