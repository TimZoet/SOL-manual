Texture Manager
===============

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    manager/images

The :code:`sol::TextureManager` class manages collections of images and textures. It provides methods to, among other
things, create new image and texture objects. The below diagram shows the relation between objects.

.. figure:: /_static/images/texture_manager.svg
    :alt: Diagram of the texture manager class structure.

To construct a new manager a fully initialized :code:`sol::MemoryManager` is needed, as this class provides e.g.
command buffers and queues:

.. code-block:: cpp

    auto memoryManager = std::make_unique<sol::MemoryManager>(...);
    auto textureManager = std::make_unique<sol::TextureManager>(*memoryManager);

One thing you'll notice is that the manager does not support loading image files from disk (or memory, for that matter).
Images can only be created by providing the actual data to be copied to the image buffers. This is by design. There are
many image loading libraries there with support for a large variety of image formats.