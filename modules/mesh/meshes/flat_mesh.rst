Flat Mesh
=========

The :code:`sol::FlatMesh` class implements the :code:`sol::IMesh` interface. It is the simplest possible mesh, holding
just a vertex buffer.

One way of creating a new :code:`sol::FlatMesh` is with a :code:`sol::MeshDescription`. This will allocate a new vertex
buffer and stage a copy that is executed when the :code:`transferStagedCopies` of the :code:`sol::MeshManager` is 
called:

.. code-block:: cpp

    // Create a new FlatMesh and automatically stage a copy:
    sol::MeshDescriptionPtr desc     = ...;
    auto&                   flatMesh = meshManager->createFlatMesh(std::move(desc));

    ...

    // Transfer data from MeshDescription to vertex buffer. flatMesh can now be rendered.
    meshManager->transferStagedCopies();

.. note::
    It is not yet possible to create a :code:`sol::FlatMesh` without using a :code:`sol::MeshDescription`.
