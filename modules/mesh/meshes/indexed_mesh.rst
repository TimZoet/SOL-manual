Indexed Mesh
============

The :code:`sol::IndexedMesh` class implements the :code:`sol::IMesh` interface. It holds a single vertex and index
buffer.

One way of creating a new :code:`sol::IndexedMesh` is with a :code:`sol::MeshDescription`. This will allocate a new
vertex and index buffer and stage a copy that is executed when the :code:`transferStagedCopies` of the
:code:`sol::MeshManager` is called:

.. code-block:: cpp

    // Create a new IndexedMesh and automatically stage a copy:
    sol::MeshDescriptionPtr desc        = ...;
    auto&                   indexedMesh = meshManager->createIndexedMesh(std::move(desc));

    ...

    // Transfer data from MeshDescription to vertex and index buffers. indexedMesh can now be rendered.
    meshManager->transferStagedCopies();

.. note::
    It is not yet possible to create a :code:`sol::IndexedMesh` without using a :code:`sol::MeshDescription`.
