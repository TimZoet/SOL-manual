Textures
========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    textures/texture1d
    textures/texture2d
    textures/texture3d
    textures/texture1darray
    textures/texture2darray
    textures/texture3darray

The :code:`sol::Texture...` classes combine an image and sampler object. They also create an image view for the image.

Textures do not own the image and sampler they reference, which allows you to reuse them between textures. For example,
you typically have only a few different samplers, while the number of images could be in the thousands.

.. code-block:: cpp
    :caption: Creating some textures.

    std::vector<sol::Image2D*>  images = ...;
    sol::Sampler2D&            sampler = ...;

    // Create a texture for each image, reusing the single sampler.
    std::vector<sol::Texture2DPtr> textures;
    for (auto* img : images)
        textures.emplace_back(sol::Texture2D::create(sol::Texture2D::Settings{
            .image = img,
            .sampler = sampler
        }));

    // Do some stuff with textures, like binding to descriptors.
    ...
