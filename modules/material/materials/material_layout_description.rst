Material Layout Description
===========================

:code:`sol::MaterialLayoutDescription` is a very simple class that holds a number of lists to define descriptor set
layouts (:code:`VkDescriptorSetLayout`). Creation of layouts is not handled by this class, it merely describes them.
Ultimately, the bindings and push constants contained in the description are passed to one or more calls of 
:code:`vkCreateDescriptorSetLayout` to create the actual handles.

As an example, we could define a simple description for some ray tracing material that is just going to write a depth
and normal to a pair of storage images:

.. code-block:: cpp

    sol::MaterialLayoutDescription desc;

    // Add a new acceleration structure binding.
    auto& as   = desc.accelerationStructures.emplace_back();
    // Add a user-friendly name. Not requird, but can be useful for debugging purposes.
    as.name    = "acceleration_structure";
    // Set and binding index must of course match what is in the shader(s).
    as.set     = 0;
    as.binding = 0;
    // And of course, the stage(s) where this binding is to be accessed must be defined.
    as.stages  = VK_SHADER_STAGE_RAYGEN_BIT_KHR;

    // Now do the same for the storage images.
    auto& d   = desc.storageImages.emplace_back();
    d.name    = "target_depth";
    d.set     = 1;
    d.binding = 0;
    d.stages  = VK_SHADER_STAGE_RAYGEN_BIT_KHR;
    // Note that the lists of bindings are std::vectors. If a reallocation takes place,
    // references to binding objects become invalid.
    auto& n   = desc.storageImages.emplace_back();
    n.name    = "target_normal";
    n.set     = 1;
    n.binding = 1;
    n.stages  = VK_SHADER_STAGE_RAYGEN_BIT_KHR;

The next step is finalizing some material layout with the description (note that this layout is probably contained in a
:code:`sol::RayTracingMaterial`):

.. code-block:: cpp

    // Use desc to finalize a sol::RayTracingMaterialLayout.
    sol::RayTracingMaterialLayout& layout = ...;
    layout.finalize(desc);

Unless we :code:`std::move` the description into the layout, it is still valid and unmodified afterwards. We can modify
it, use it to finalize another layout (even a non-ray tracing layout), etc.

.. note::

    TODO: Push constants.
