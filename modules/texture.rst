sol.texture
===========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    texture/images
    texture/samplers
    texture/textures

The :code:`sol.texture` module contains functionality to manage image, sampler and texture resources.

With Vulkan, the management of image resources is quite complex. Image views, samplers, layout transitions, queue family
ownership transfers, and of course actually transferring data to image buffers require a lot of work. The classes in
this module do a lot of the heavy lifting, while still offering a lot of control over what happens when.
