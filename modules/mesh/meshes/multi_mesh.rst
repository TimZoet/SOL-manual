Multi Mesh
==========

The :code:`sol::MultiMesh` class implements the :code:`sol::IMesh` interface. It can hold one or more vertex buffers and
optionally an index buffer. As such, it essentially offers a superset of the features of the :code:`sol::FlatMesh` and
:code:`sol::IndexedMesh`.

One way of creating a new :code:`sol::MultiMesh` is with a :code:`sol::MeshDescription`. This will allocate the required
vertex and index buffers and stage a copy that is executed when the :code:`transferStagedCopies` of the
:code:`sol::MeshManager` is called:

.. code-block:: cpp

    // Create a new MultiMesh and automatically stage a copy:
    sol::MeshDescriptionPtr desc      = ...;
    auto&                   multiMesh = meshManager->createMultiMesh(std::move(desc));

    ...

    // Transfer data from MeshDescription to vertex and index buffers. multiMesh can now be rendered.
    meshManager->transferStagedCopies();

.. note::
    It is not yet possible to create a :code:`sol::MultiMesh` without using a :code:`sol::MeshDescription`.
