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
    :caption: Creating and destroying some samplers.

    sol::TextureCollection& collection = ...;

    // Create a number of samplers.
    auto& samplerA = collection.createSampler2D();
    auto& samplerB = collection.createSampler3D(VK_FILTER_NEAREST, VK_FILTER_NEAREST);

    // Do some stuff with samplers, like making textures.
    ...

    // Deallocate once they're no longer in use.
    // Note: Calling any method on destroyed samplers is undefined behaviour.
    collection.destroySampler(samplerA);
    collection.destroySampler(samplerB);
