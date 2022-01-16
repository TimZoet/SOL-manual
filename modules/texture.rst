sol.texture
===========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    texture/texture_manager
    texture/image_transfer
    texture/image2d
    texture/texture2d

The :code:`sol.texture` module contains functionality to manage image and texture resources. At the center lies the
:code:`sol::TextureManager`, which defines the main interface to image and texture creation.

.. figure:: /_static/images/texture_manager.svg
    :alt: Diagram of the texture manager class structure.

With Vulkan, the management of image resources is quite complex. Image views, samplers, layout transitions, queue family
ownership transfers, and of course actually transferring data to image buffers require a lot of work. The classes in
this module do a lot of the heavy lifting, while still offering full control over what happens.
