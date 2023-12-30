Samplers
========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    samplers/sampler1d
    samplers/sampler2d
    samplers/sampler3d

The :code:`sol::Sampler...` classes manage a sampler object. Since samplers cannot be modified after creation, all their
parameters are specified on creation.

.. code-block:: cpp
    :caption: Creating some samplers.

    sol::VulkanDevice& device = ...;

    // Create a number of samplers.
    auto samplerA = sol::Sampler2D::create(sol::Sampler2D::Settings{
        .device = device,
        .magFilter = VK_FILTER_NEAREST,
        ...
    });
    auto samplerB = sol::Sampler3D::create(sol::Sampler3D::Settings{
        .device = device,
        .magFilter = VK_FILTER_NEAREST,
        ...
    });

    // Do some stuff with samplers, like making textures.
    ...
