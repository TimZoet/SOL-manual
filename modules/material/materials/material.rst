Material
========

The :code:`sol::Material` class is the base class of all pipeline specific (i.e. compute, graphics and ray tracing)
material classes. On its own, it doesn't do very much. It really is the deriving classes that add most features.
However, it is important to understand how flexible the material system is. There is not a single best way to define
classes that applies to all use-cases.

Let's study the following example. It represents a material that will draw all objects with a simple stripe pattern that
alternates between 2 colors:

.. code-block:: cpp

    class StripedMaterial : public sol::GraphicsMaterial
    {
    public:
        StripedMaterial(sol::MeshLayout&         meshLayout,
                        sol::VulkanShaderModule& vertexModule,
                        sol::VulkanShaderModule& fragmentModule) :
            GraphicsMaterial(vertexModule,
                             fragmentModule)
        {
            // Set mesh layout.
            layout.setMeshLayout(meshLayout);

            // Define (dynamic) states.
            layout.enableDynamicState<VK_DYNAMIC_STATE_VIEWPORT>(true);
            layout.setDynamicViewportCount(1);
            layout.enableDynamicState<VK_DYNAMIC_STATE_SCISSOR>(true);
            layout.setDynamicScissorCount(1);
            layout.setCullMode(sol::CullMode::None);

            // Define color blending.
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

            // Add all descriptor sets.
            sol::MaterialLayoutDescription desc;
            auto& vp        = desc.uniformBuffers.emplace_back();
            vp.name         = "ViewProjection";
            vp.set          = 0;
            vp.binding      = 0;
            vp.count        = 2;                     // 2
            vp.size         = sizeof(math::mat4x4f); // matrices for the camera
            vp.stages       = VK_SHADER_STAGE_VERTEX_BIT;
            auto& stripes   = desc.uniformBuffers.emplace_back();
            stripes.name    = "StripeParameters";
            stripes.set     = 1;
            stripes.binding = 0;
            stripes.count   = 1;
            stripes.size    = sizeof(float) * 8; // orientation, scale, and 2 rgb colors to alternate between
            stripes.stages  = VK_SHADER_STAGE_FRAGMENT_BIT;
            auto& model     = desc.pushConstants.emplace_back();
            model.name      = "Model";
            model.offset    = 0;
            model.size      = sizeof(math::mat4x4f); // transformation matrix of the object
            model.stages    = VK_SHADER_STAGE_VERTEX_BIT;

            // Finalize at end of constructor.
            layout.finalize(std::move(desc));
        }
    };

    auto mtl = std::make_unique<StripedMaterial>(getMeshLayoutFromSomewhere(),
                                                 loadShader("default.vert.spv"),
                                                 loadShader("default.frag.spv"));

As you can see, the material is defined entirely in the constructor. While in some scenarios this may be useful, here
some redundancy can be found. Just imagine if we wanted to create another material that rendered dots. It would repeat
almost all code!

Color Blending
--------------

Firstly, let's improve setting up the color blending options. In a real life scenario, we probably have only a few 
distinct options. One for opaque materials, another for transparency, and maybe something for emissive objects. That can
all be put into several utility functions that we call inside of the constructors:

.. code-block:: cpp

    namespace utils
    {
        void setOpaqueColorBlending(sol::GraphicsMaterialLayout& layout)
        {
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
        }
    }

    StripedMaterial::StripedMaterial(...)
    {
        ...
        utils::setOpaqueColorBlending(layout);
        ...
        layout.finalize(...);
    }

We could have gone another route as well. For example, by inserting a new class, e.g. :code:`OpaqueMaterial`, in
between our :code:`StripedMaterial` and :code:`sol::GraphicsMaterial` that takes care of setting up the blending. The
dotted material could inherit from the opaque material as well.

Layout Description
------------------

Setting up the material layout description is going to be almost identical for all our materials. All of them will have
a view and projection matrix, and most of them are going to have a transformation matrix per object that is being drawn.
We can again extract setting up the shared code, and then add some specific code in the class. To make a follow-up
example a bit shorter, we'll be putting the specific code in a separate, public method called
:code:`setupDescriptorSet` in advance:

.. code-block:: cpp

    namespace utils
    {
        sol::MaterialLayoutDescription createDefaultMaterialLayoutDescription()
        {
            sol::MaterialLayoutDescription desc;
            auto& vp     = desc.uniformBuffers.emplace_back();
            vp.name      = "ViewProjection";
            vp.set       = 0;
            vp.binding   = 0;
            vp.count     = 2;                     // 2
            vp.size      = sizeof(math::mat4x4f); // matrices for the camera
            vp.stages    = VK_SHADER_STAGE_VERTEX_BIT;
            auto& model  = desc.pushConstants.emplace_back();
            model.name   = "Model";
            model.offset = 0;
            model.size   = sizeof(math::mat4x4f); // transformation matrix of the object
            model.stages = VK_SHADER_STAGE_VERTEX_BIT;
            return desc;
        }
    }

    StripedMaterial::StripedMaterial(..., sol::MaterialLayoutDescription desc)
    {
        ...

        // Add the material specific descriptor set.
        setupDescriptorSet(desc);

        layout.finalize(std::move(desc));
    }

    void StripedMaterial::setupDescriptorSet(sol::MaterialLayoutDescription& desc)
    {
        auto& stripes   = desc.uniformBuffers.emplace_back();
        stripes.name    = "StripeParameters";
        stripes.set     = 1;
        stripes.binding = 0;
        stripes.count   = 1;
        stripes.size    = sizeof(float) * 8; // orientation, scale, and 2 rgb colors to alternate between
        stripes.stages  = VK_SHADER_STAGE_FRAGMENT_BIT;
    }
    
    auto mtl = std::make_unique<StripedMaterial>(..., createDefaultMaterialLayoutDescription());

Shaders
-------

It is not written anywhere that the material must know which shaders it will be using before pipelines are created from
it. The layout does not require them. The shader module parameters can be ripped out entirely from the constructor. One
thing we do have to add now is a :code:`sol::VulkanDevice` parameter. The material base class needs it:

.. code-block:: cpp

    StripedMaterial::StripedMaterial(sol::VulkanDevice& device, ...) 
        : GraphicsMaterial(device)
    {
        ...
    }
    
    auto mtl = std::make_unique<StripedMaterial>(getDevice(), ...);
    // Set shaders from outside of class. As long as it happens before creating a pipeline.
    mtl->setVertexShader(loadShader("default.vert.spv"));
    mtl->setFragmentShader(loadShader("default.frag.spv"));

Note that you should probably take care when doing this. Modifying the assigned shaders after already having created
pipelines could cause confusion later down the line.

(Dynamic) State and Mesh Layout
-------------------------------

The remaining setup for (dynamic) state and mesh layout could be treated in the same manner as the color blending.
Alternatively, we could move them out of the constructor entirely. In order to do that, we will also have to take out
finalization of the layout, as well as the material specific descriptor set. This is simple, because the layout is
publicly accessible. The full implementation will now more or less look like this:

.. code-block:: cpp

    namespace utils
    {
        void setDefaultGraphicsPipelineStates(sol::GraphicsMaterialLayout& layout)
        {
            layout.enableDynamicState<VK_DYNAMIC_STATE_VIEWPORT>(true);
            layout.setDynamicViewportCount(1);
            layout.enableDynamicState<VK_DYNAMIC_STATE_SCISSOR>(true);
            layout.setDynamicScissorCount(1);
            layout.setCullMode(sol::CullMode::None);
        }
    }

    StripedMaterial::StripedMaterial(sol::VulkanDevice& device) : GraphicsMaterial(device)
    {
        utils::setOpaqueColorBlending(layout);
    }
    
    auto mtl  = std::make_unique<StripedMaterial>(getDevice());
    auto desc = createDefaultMaterialLayoutDescription();
    mtl->setupDescriptorSet(desc);
    setDefaultGraphicsPipelineStates(mtl->getGraphicsLayout());
    mtl->setMeshLayout(getMeshLayoutFromSomewhere());
    mtl->getGraphicsLayout().finalize(std::move(desc));
    mtl->setVertexShader(loadShader("default.vert.spv"));
    mtl->setFragmentShader(loadShader("default.frag.spv"));

Conclusion
----------

Again, the material system is very flexible. All examples found above are mere suggestions. You get to decide how you
organize your code exactly. You could even take things to an extreme and never create new classes, using the graphics
(and compute and ray tracing) material classes directly.
