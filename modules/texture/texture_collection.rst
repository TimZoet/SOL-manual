Texture Collection
==================

The :code:`sol::TextureCollection` class manages a collection of :doc:`/modules/texture/images`,
:doc:`/modules/texture/samplers` and :doc:`/modules/texture/textures`. It has a simple interface for creating and
destroying these objects.

.. code-block:: cpp
    :caption: Creating a texture collection.

    sol::MemoryManager&          manager = ...;
    sol::TextureCollectionPtr collection = std::make_unique<sol::TextureCollection>(manager);

You can create any number of texture collections, each managing their own set of resources. This can be useful to keep
different parts of your application completely independent, and simplify the lifetime management of sets of related
resources. Also, a single texture collection is not thread safe. Using multiple collections can help avoid expensive
locking mechanisms.
