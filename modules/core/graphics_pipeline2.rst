Graphics Pipeline 2
===================

The :code:`sol::VulkanGraphicsPipeline2` class manages the lifetime of a :code:`VkPipeline`. It will eventually replace
:doc:`/modules/core/graphics_pipeline`.

Currently, the only way to create a pipeline is by linking together pipeline libraries.

.. code-block:: cpp

    // Assuming previously created pipeline libraries.
    sol::VulkanGraphicsPipelineVertexInput&       vertexInputLib = ...;
    sol::VulkanGraphicsPipelinePreRasterization&      preRastLib = ...;
    sol::VulkanGraphicsPipelineFragment&             fragmentLib = ...;
    sol::VulkanGraphicsPipelineFragmentOutput& fragmentOutputLib = ...;

    // Create a pipeline by linking together the libraries.
    sol::VulkanGraphicsPipeline2::Settings settings;
    settings.device                   = vertexInputLib.getDevice();
    settings.vertexInputPipeline      = vertexInputLib;
    settings.preRasterizationPipeline = preRastLib;
    settings.fragmentPipeline         = fragmentLib;
    settings.fragmentOutputPipeline   = fragmentOutputLib;
    
    auto graphicsPipeline = sol::VulkanGraphicsPipeline2::create(setting);
