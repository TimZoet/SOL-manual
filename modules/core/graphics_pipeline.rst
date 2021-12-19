Graphics Pipeline
=================

The :code:`sol::VulkanGraphicsPipeline` class manages the lifetime of a :code:`VkPipeline` (and 
:code:`VkPipelineLayout`). Since pipelines are highly configurable, there are many settings.

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::VulkanRenderPass& renderPass       = ...;
    sol::VulkanShaderModule& vertexShader   = ...;
    sol::VulkanShaderModule& fragmentShader = ...;

    // Create a pipeline with vertex and fragment shader.
    sol::VulkanGraphicsPipeline::Settings settings;
    settings.renderPass = renderPass;
    settings.extent           = extent;
    settings.vertexShader     = vertexShader;
    settings.fragmentShader   = fragmentShader;
    settings.vertexAttributes = ...;
    settings.vertexBindings   = ...;
    ...

    auto graphicsPipeline = sol::VulkanGraphicsPipeline::create(setting);

.. note::
    Many features for pipeline creation are not yet exposed.
