Compute Pipeline
================

The :code:`sol::VulkanComputePipeline` class manages the lifetime of a :code:`VkPipeline` (and 
:code:`VkPipelineLayout`).

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::VulkanShaderModule& computeShader                 = ...;
    std::vector<sol::VulkanDescriptorSetLayoutPtr> layouts = ...;

    // Create a pipeline.
    sol::VulkanComputePipeline::Settings settings;
    settings.computeShader        = computeShader;
    settings.descriptorSetLayouts = layouts;
    ...

    auto computePipeline = sol::VulkanComputePipeline::create(setting);

.. note::

    It is not yet possible to specify the name of the entry point.
