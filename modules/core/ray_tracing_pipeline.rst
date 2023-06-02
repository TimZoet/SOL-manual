Ray Tracing Pipeline
====================

The :code:`sol::VulkanRayTracingPipeline` class manages the lifetime of a :code:`VkPipeline` (and 
:code:`VkPipelineLayout`).

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::VulkanShaderModule& raygen                        = ...;
    sol::VulkanShaderModule& anyHit                        = ...;
    sol::VulkanShaderModule& closestHit                    = ...;
    sol::VulkanShaderModule& miss                          = ...;
    std::vector<sol::VulkanDescriptorSetLayoutPtr> layouts = ...;

    // Create a pipeline.
    sol::VulkanRayTracingPipeline::Settings settings;
    settings.raygenShader         = raygen;
    settings.missShader           = miss;
    settings.closestHitShader     = closestHit;
    settings.anyHitShader         = anyHit;
    settings.descriptorSetLayouts = layouts;
    ...

    auto rtPipeline = sol::VulkanRayTracingPipeline::create(setting);

.. note::

    Callables are not supported yet.

.. note::

    It is not yet possible to specify the name of the entry point for each stage.
