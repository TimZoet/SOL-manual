Ray Tracing Pipeline
====================

The :code:`sol::VulkanRayTracingPipeline` class manages the lifetime of a :code:`VkPipeline` (and 
:code:`VkPipelineLayout`).

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::VulkanShaderModule& raygen                        = ...;
    sol::VulkanShaderModule& miss                          = ...;
    sol::VulkanShaderModule& closestHit                    = ...;
    std::vector<sol::VulkanDescriptorSetLayoutPtr> layouts = ...;

    // Create a pipeline.
    sol::VulkanRayTracingPipeline::Settings settings;
    settings.raygenShader          = raygen;
    settings.missShaders          += miss;
    settings.closestHitShaders    += closestHit;
    settings.descriptorSetLayouts  = layouts;
    ...

    auto rtPipeline = sol::VulkanRayTracingPipeline::create(setting);

Shader Stages
-------------

You must always specify the :code:`raygenShader`. Optionally, an entry point can be set. If not set, it defaults to
:code:`"main"`.

.. code-block:: cpp

    sol::VulkanShaderModule& raygen = ...;
    settings.raygenShader     = raygen;
    settings.raygenEntryPoint = "other";

You can specify a list of one or more :code:`missShaders`. Optionally, an entry point for each shader can be set. If not
set, it defaults to :code:`"main"`.

.. code-block:: cpp

    std::vector<sol::VulkanShaderModule*> miss = ...;

    settings.missShaders = miss;
    for (size_t i = 0; i < miss.size(); i++)
        settings.missEntryPoints.push_back(std::string("miss_") + std::to_string(i));

You can specify a list of zero or more :code:`closestHitShaders`. Optionally, an entry point for each shader can be set.
If not set, it defaults to :code:`"main"`.

.. code-block:: cpp

    std::vector<sol::VulkanShaderModule*> closest = ...;

    settings.closestHitShaders = closestHit;
    for (size_t i = 0; i < closestHit.size(); i++)
        settings.closestHitEntryPoints.push_back(std::string("closest_") + std::to_string(i));

You can specify a list of zero or more :code:`anyHitShaders`. Optionally, an entry point for each shader can be set.
If not set, it defaults to :code:`"main"`.

.. code-block:: cpp

    std::vector<sol::VulkanShaderModule*> anyHit = ...;

    settings.anyHitShaders = anyHit;
    for (size_t i = 0; i < anyHit.size(); i++)
        settings.anyHitEntryPoints.push_back(std::string("any_") + std::to_string(i));

You can specify a list of zero or more :code:`intersectionShaders`. Optionally, an entry point for each shader can be set.
If not set, it defaults to :code:`"main"`.

.. code-block:: cpp

    std::vector<sol::VulkanShaderModule*> intersection = ...;

    settings.intersectionShaders = intersection;
    for (size_t i = 0; i < intersection.size(); i++)
        settings.intersectionEntryPoints.push_back(std::string("intersection_") + std::to_string(i));

You can specify a list of zero or more:code:`callableShaders`. Optionally, An entry point for each shader can be set. If
not set, it defaults to :code:`"main"`.

.. code-block:: cpp

    std::vector<sol::VulkanShaderModule*> callables = ...;

    settings.callableShaders = callables;
    for (size_t i = 0; i < callables.size(); i++)
        settings.callableEntryPoints.push_back(std::string("func_") + std::to_string(i));

The shader stages are laid out in a fixed manner to simplify the calculation of the indices for the shader groups. See
below diagram.

.. figure:: /_static/images/core/rt_shader_stages.svg
    :alt: Layout of the shader stages.

Shader Groups
-------------

With the exception of hit groups, all shader groups are generated automatically based on the provided shader stages.

Hit groups have to be specified explicitly using triplets of indices for closest hit, any hit and intersection shaders.
The indices for each type of hit/intersection shader are relative to the number of stages for that particular type, and
are therefore not affected by how many shader stages for other types there are. The proper, absolute indices are
calculated automatically.

.. code-block:: cpp

    // Assuming there are 1 closest hit, 3 any hit and 2 intersection shaders...
    settings.closestHitShaders   = {...};
    settings.anyHitShaders       = {..., ..., ...};
    settings.intersectionShaders = {..., ...};
    // Valid index ranges are:
    // [0] for closest hit,
    // [0, 1, 2] for any hit,
    // [0, 1] for intersection.
    settings.hitGroups.emplace_back(0, 0, std::nullopt);
    settings.hitGroups.emplace_back(0, 1, std::nullopt);
    settings.hitGroups.emplace_back(0, 2, 0);
    settings.hitGroups.emplace_back(0, 2, 1);
