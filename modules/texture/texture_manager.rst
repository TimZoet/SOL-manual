Texture Manager
===============

The :code:`sol::TextureManager` class manages collections of images and textures. It provides methods to, among other
things, create new image and texture objects. To construct a new manager a fully initialized :code:`sol::MemoryManager`
is needed, as this class provides e.g. command buffers and queues:

.. code-block:: cpp

    auto memoryManager = std::make_unique<sol::MemoryManager>(...);
    auto textureManager = std::make_unique<sol::TextureManager>(*memoryManager);

It is important to note that it is possible to create more than one manager, managing distinct sets of resources. There
are many reasons you might do this. Most importantly, it allows you to keep fine grained control over when resources are
created, updated, used, and destroyed. Apart from that, it can also be convenient. You could for example have one
manager for resources that are completely static during the lifetime of your program, and another for the more volatile
textures and images.

One thing you'll notice is that the manager does not support loading image files from disk (or memory, for that matter).
Images can only be created by providing the actual data to be copied to the image buffers. This is by design. There are
many image loading libraries out there with support for a large variety of image formats.