Texture Manager
===============

The :code:`sol::TextureManager` class manages collections of images and textures. It provides methods to, among other
things, create new image and texture objects. The below diagram shows the relation between objects.

.. figure:: /_static/images/texture_manager.svg
    :alt: Diagram of the texture manager class structure.

The :code:`sol::TextureManager` itself is owned by the :code:`sol::MemoryManager`, and relies on that class for several 
things related to the management of memory, buffers, etc.

One thing you'll notice is that the manager does not support loading image files from disk (or memory, for that matter).
Images can only be created by providing the actual data to be copied to the image buffers. This is by design. The 
manager and related classes are kept as simple as possible.