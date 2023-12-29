Graphics Material
=================

The :code:`sol::GraphicsMaterial` class inherits from the :code:`sol::Material` class. It represents a complete graphics
pipeline from which you can create instances.

Instances
---------

Material instances can be constructed directly from a material.

.. code-block:: cpp
    :caption: Creating instances of a graphics material.

    sol::GraphicsMaterial& mtl = ...;

    auto inst0 = mtl.createInstance<sol::GraphicsMaterialInstance>();

    // Any class that derives from sol::GraphicsMaterialInstance is supported.
    // Additional arguments are forwarded to the constructor.
    auto inst1 = mtl.createInstance<MyMaterialInstanceClass>(10.0f, textureObj);

Dynamic State
-------------

On construction, the material will inspect the pipeline from which it was created to see which dynamic states were
enabled. You can query the material for the set of enabled states using :code:`getDynamicStates` or directly check if a
specific state is enabled by calling :code:`isDynamicStateEnabled`.

For each type of dynamic state, there is a dedicated class that inherits from :code:`sol::GraphicsDynamicState`. You can
create instances of these classes yourself or get the material to do it by calling the templated
:code:`createDynamicState` method. This method will verify that the requested dynamic state was actually enabled for the
material, although the state objects are not explicitly linked to any material object:

.. code-block:: cpp
    :caption: Creating dynamic states for a graphics material.

    // Assuming a material that has a pipeline created with
    // VK_DYNAMIC_STATE_SCISSOR_WITH_COUNT and VK_DYNAMIC_STATE_VIEWPORT_WITH_COUNT.
    sol::GraphicsMaterial& mtl = ...;

    // You can create dynamic states directly from the material...
    auto scissor = mtl.createDynamicState<sol::Scissor>();
    // ...or just through the constructor.
    auto viewport = std::make_unique<sol::Viewport>();

    // This would throw, because the pipeline did not 
    mtl.createDynamicState<sol::PolygonMode>();

.. note:: 

    Far from all dynamic states have been implemented yet.
