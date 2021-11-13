Render Pass
===========

The :code:`sol::VulkanRenderPass` manages the lifetime of a :code:`VkRenderPass`. The most important creation parameter is of
course its layout. For this there is the :code:`VulkanRenderPassLayout`, which is created independently and can be 
reused. Through the layout all attachments, subpasses and their dependencies are defined.

.. code-block:: cpp

    // Specify attachments, subpasses, dependencies.
    sol::VulkanRenderPassLayout layout;
    ...
    layout.finalize();

    // Create actual render pass.
    sol::VulkanRenderPass::Settings settings;
    settings.device = device;
    settings.layout = &layout;
    auto renderPass = sol::VulkanRenderPass::create(settings);

Attachments
-----------

New attachments can be added to a layout by calling :code:`createAttachment`. This returns a reference to the newly 
created :code:`sol::VulkanAttachment`, which can then be configured. For a basic rendering pipeline using 8-bit RGB 
colors and 32-bit depth, something like the following could be done:

.. code-block:: cpp

    auto& colorAttachment = layout.createAttachment();
    colorAttachment.setFormat(VK_FORMAT_R8G8B8_SRGB);
    colorAttachment.setSamples(VK_SAMPLE_COUNT_1_BIT);
    colorAttachment.setLoadOp(VK_ATTACHMENT_LOAD_OP_CLEAR);
    colorAttachment.setStoreOp(VK_ATTACHMENT_STORE_OP_STORE);
    colorAttachment.setStencilLoadOp(VK_ATTACHMENT_LOAD_OP_DONT_CARE);
    colorAttachment.setStencilStoreOp(VK_ATTACHMENT_STORE_OP_DONT_CARE);
    colorAttachment.setInitialLayout(VK_IMAGE_LAYOUT_UNDEFINED);
    colorAttachment.setFinalLayout(VK_IMAGE_LAYOUT_PRESENT_SRC_KHR);

    auto& depthAttachment = layout.createAttachment();
    depthAttachment.setFormat(VK_FORMAT_D32_SFLOAT);
    depthAttachment.setSamples(VK_SAMPLE_COUNT_1_BIT);
    depthAttachment.setLoadOp(VK_ATTACHMENT_LOAD_OP_CLEAR);
    depthAttachment.setStoreOp(VK_ATTACHMENT_STORE_OP_DONT_CARE);
    depthAttachment.setStencilLoadOp(VK_ATTACHMENT_LOAD_OP_DONT_CARE);
    depthAttachment.setStencilStoreOp(VK_ATTACHMENT_STORE_OP_DONT_CARE);
    depthAttachment.setInitialLayout(VK_IMAGE_LAYOUT_UNDEFINED);
    depthAttachment.setFinalLayout(VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL);

For ease of use, attachments are copyable, both within the same layout and between layouts:

.. code-block:: cpp

    auto& colorAttachment2 = someOtherLayout.createAttachment();
    colorAttachment2       = colorAttachment;
    colorAttachment2.setFinalLayout(VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL);

Subpasses
---------

New subpasses can be added to a layout by calling :code:`createSubpass`. This returns a reference to the newly created
:code:`sol::VulkanSubpass`, which can then be configured. Naturally, the order in which subpasses are created is their
order in the render pass. Continuing the example above:

.. code-block:: cpp

    auto& subpass = layout.createSubpass();
    subpass.setPipelineBindPoint(VK_PIPELINE_BIND_POINT_GRAPHICS);
    subpass.addColorAttachment(colorAttachment, VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL);
    subpass.setDepthStencilAttachment(depthAttachment, VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL);

Besides color and depth/stencil attachments, you can of course also configure all other types of attachments using the
appropriate methods.

Note that while subpasses are copyable, there are some limitations. The properties of the subpass itself (e.g. pipeline
bind point) are always copied. Attachments and subpass dependencies are only copied when the other subpass is in the 
same layout, but ignored when two different layouts are involved.

Subpass Dependencies
--------------------

There are two methods to add dependencies. The first, :code:`addExternalDependency` adds an external dependency, with 
the subpass on which the method is called being treated as either the source or destination. The additional parameters 
specify the pipeline stage and access flags. Again, creating a possible dependency for the previous example:

.. code-block:: cpp

    subpass.addExternalDependency(
        true,
        VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT,
        0,
        VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT,
        VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT | VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT,
        0
    );

:code:`addDependency` does the same, but with a subpass within the render pass. The subpass parameter is used as the 
source. The remaining parameters are the same:

.. code-block:: cpp

    auto& subpass0 = layout.createSubpass();
    auto& subpass1 = layout.createSubpass();
    subpass1.addDependency(subpass0, ...);

Visualization
-------------

For debugging purposes the layout can be visualized as a :code:`dot` graph:

.. code-block:: cpp
    
    dot::Graph dotGraph;
    renderPassLayout.visualize(dotGraph);

    std::ofstream dgf("renderpass.dot");
    dotGraph.write(dgf);

.. figure:: /_static/images/render_pass_layout.svg
    :alt: Diagram of a render pass layout.
