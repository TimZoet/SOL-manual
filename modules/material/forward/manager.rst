Forward Material Manager
========================

The :code:`sol::ForwardMaterialManager` class manages collections of forward material instances and graphics pipelines
created from forward materials. It also keeps track of uniform buffers and descriptor sets. Renderers make use of this
class when recording command buffers. The below diagram shows the relation between objects.

.. note::

    TODO: Create diagram. Recall (somewhat outdated) forward_material_data.svg.

To construct a new manager a fully initialized :code:`sol::MemoryManager` is needed, as this class provides e.g. methods
for uniform buffer allocation:

.. code-block:: cpp

    auto memoryManager = std::make_unique<sol::MemoryManager>(...);
    auto fwdMtlManager = std::make_unique<sol::ForwardMaterialManager>(*memoryManager);

Creating Material Instances
---------------------------

To create a new material instance, you need at least a class deriving from :code:`sol::ForwardMaterial` (see 
:doc:`material`) and a matching class deriving from :code:`sol::ForwardMaterialInstance` (see :doc:`instance`). With a 
material object as the first parameter you can then call :code:`createMaterial<YourMaterialInstanceClass>` to create a
material instance of the given type. Any additional parameters are forwarded to the constructor of the instance class:

.. code-block:: cpp

    // Assuming some ForwardMaterial class and an object of that type.
    class MyMaterial : public sol::ForwardMaterial {...};
    auto myMtl = std::make_unique<MyMaterial>(...);

    // Assuming a ForwardMaterialInstance class for the above ForwardMaterial.
    class MyMaterialInstance : public sol::ForwardMaterialInstance {...};

    // Create a new material instance of MyMaterialInstance.
    // Requires a reference to the material object and whatever parameters the constructor takes.
    auto& myMtlInstance = fwdMtlManager->createMaterial<MyMaterialInstance>(*myMtl, ...);

Note that while the manager does own the material instance, it does not take ownership of the material object. It is
your responsibility to make sure that the material objects exists as long as the manager exists.

Creating Pipelines
------------------

After creating material instances, but before rendering where inevitably one or more pipelines need to be bound, these 
pipelines must be created. For performance reasons, different instances of the same forward material can and should
share their pipeline as often as possible. There are 3 objects that influence pipeline creation:

Firstly, the (object deriving from) :code:`sol::ForwardMaterial` itself. As you will see in the sections next to this 
one, it contains the entire material layout, mesh layout, shader modules, etc. These are some of the most important
properties of a pipeline.

Secondly, the :code:`sol::RenderSettings`. This object contains many properties that impact pipeline creation as well.
For example, the culling mode. Two different renderers can share settings, or have completely different settings
requiring different pipelines even when rendering the same material.

Thirdly, the :code:`sol::VulkanRenderPass`. Each pipeline must be created with a compatible render pass.

So, if we have two materials that we want to render using some combination of settings and render pass, we could do the
following:

.. code-block:: cpp

    // Assuming settings, render pass, and of course a manager.
    sol::RenderSettings&          settings = ...;
    sol::VulkanRenderPass&      renderPass = ...;
    sol::ForwardMaterialManagerPtr manager = ...;

    // We have two materials for which we want to create a pipeline.
    MyMaterial&          myMtl = ...;
    SomeOtherMaterial& someMtl = ...;
    manager->createPipeline(myMtl, settings, renderPass);
    manager->createPipeline(someMtl, settings, renderPass);

Creation of pipelines you will typically do between traversal of your scenegraph(s), as that tells you which materials
will need a pipeline, and running a renderer, which you do with specific settings and a render pass. Once you have
ensured all necessary pipelines have been constructed, you can safely invoke the renderer. Inside of a typical renderer
you will probably find something such as this:

.. code-block:: cpp

    // Begin a render pass, probably using the one from before.
    vkCmdBeginRenderPass(...);

    // Let us pretend we have a list of drawables that contain a pointer to a material instance
    // with descriptor sets, as well as some geometry to bind and draw.
    for (auto& drawable : drawables)
    {
        sol::ForwardMaterial&     mtlInstance = *drawable.materialInstance;
        sol::VulkanGraphicsPipeline& pipeline = manager->getPipeline(
            mtlInstance.getForwardMaterial(), settings, renderPass
        );
        
        // Bind the retrieved pipeline (let's ignore unnecessary rebinding for now).
        vkCmdBindPipeline(pipeline...)

        // Bind the descriptor sets (same here).
        vkCmdBindDescriptorSets(mtlInstance...)

        // And then do all your drawing.
        vkCmdBindVertexBuffers(...);
        vkCmdBindIndexBuffer(...);
        vkCmdDrawIndexed(...);
    }

    vkCmdEndRenderPass(...);

.. note::
    
    Currently, the manager only uses the address of both the :code:`sol::RenderPass` and :code:`sol::RenderSettings` to
    manage the pipelines for a single forward material. Most functionality when it comes to recreating pipelines has yet
    to be implemented.
