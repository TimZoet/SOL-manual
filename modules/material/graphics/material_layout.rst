Graphics Material Layout
========================

The :code:`sol::GraphicsMaterialLayout` class inherits from :code:`sol::MaterialLayout`. It adds a number of additional
features required to create a proper graphics pipeline. All values must be set before finalization of the layout.

.. code-block:: cpp

    sol::GraphicsMaterialLayout& layout = ...;

    layout.setCullMode(sol::CullMode::Back);
    layout.setFrontFace(sol::FrontFace::CounterClockwise);

Mesh Layout
-----------

The material layout must be given a :code:`sol::MeshLayout` that is compatible with the vertex shader's definition:

.. code-block:: cpp

    sol::MeshLayout& meshLayout = ...;
    layout.setMeshlayout(meshLayout);

Color Blending
--------------

Blending for color attachments can be defined through the :code:`addColorBlendAttachment` method. A typical setup with
just one color attachment may be defined as follows:

.. code-block:: cpp

    VkPipelineColorBlendAttachmentState colorBlending;
    colorBlending.blendEnable         = VK_TRUE;
    colorBlending.srcColorBlendFactor = VK_BLEND_FACTOR_SRC_ALPHA;
    colorBlending.dstColorBlendFactor = VK_BLEND_FACTOR_ONE_MINUS_SRC_ALPHA;
    colorBlending.colorBlendOp        = VK_BLEND_OP_ADD;
    colorBlending.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
    colorBlending.dstAlphaBlendFactor = VK_BLEND_FACTOR_ONE_MINUS_SRC_ALPHA;
    colorBlending.alphaBlendOp        = VK_BLEND_OP_ADD;
    colorBlending.colorWriteMask      = VK_COLOR_COMPONENT_R_BIT |
                                        VK_COLOR_COMPONENT_G_BIT |
                                        VK_COLOR_COMPONENT_B_BIT |
                                        VK_COLOR_COMPONENT_A_BIT;
    
    layout.addColorBlendAttachment(colorBlending);

Dynamic State
-------------

Some graphics pipeline state can be made dynamic. In that case, the value stored by the material layout (e.g. the cull
mode) is ignored and instead material instances will be allowed to specify dynamic state values. To enable any kind of
dynamic state, the :code:`enableDynamicState` method can be called with the matching enum value:

.. code-block:: cpp

    layout.enableDynamicState<VK_DYNAMIC_STATE_VIEWPORT>(true);
    layout.setDynamicViewportCount(1);

    layout.enableDynamicState<VK_DYNAMIC_STATE_SCISSOR>(true);
    layout.setDynamicScissorCount(1);

.. note::
    
    Not all dynamic state types have been exposed yet.
