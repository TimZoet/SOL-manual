Uniform Buffers
===============

Uniform buffer data can have many different purposes. Not only that, the size of a buffer can vary wildly, as well as 
e.g. the frequency with which it needs to be updated, or whether or not it can be shared between materials. To further 
illustrate this, consider the following use cases one might encounter in a typical renderer:

* **Projection matrix.** Small amount of data. Probably updated every frame, or whenever the camera moves. Could be 
  shared between all materials.
* **Material color.** Several bytes of data. May be updated frequently, or never at all. Unique to a material instance.
* **Animation data.** Could be anywhere from several bytes to megabytes. Probably reused between multiple objects, for 
  example when there are multiple human characters with a walk animation.

These examples just barely scratch the surface of what you might be doing. Clearly, all of this variation must be dealt
with in order to optimize rendering speed and memory usage.

A Basic Example
---------------

Before defining the more advanced settings of a uniform buffer, its basic properties such as set and binding index, 
size, and stage accessibility must be configured. Consider a forward material with the following shaders:

.. code-block:: c

    // Vertex shader:

    layout(set = 0, binding = 0) uniform UBO0
    {
        mat4 value;
    } ubo0[3];

    ...

    // Fragment shader:

    layout(set = 1, binding = 0) uniform UBO1
    {
        vec4 value;
    } ubo1;

    layout(set = 1, binding = 1) uniform UBO2
    {
        vec4 value0;
        vec4 value1;
    } ubo2;

    ...

This material's layout can be defined as follows:

.. code-block:: cpp

    MyMaterial::MyMaterial(...)
    {
        auto& ub0 = layout.addUniformBuffer();
        ub0.set     = 0;
        ub0.binding = 0;
        ub0.size    = sizeof(float) * 16;
        ub0.count   = 3;
        ub0.stages  = VK_SHADER_STAGE_VERTEX_BIT;

        auto& ub1 = layout.addUniformBuffer();
        ub1.set     = 1;
        ub1.binding = 0;
        ub1.size    = sizeof(float) * 4;
        ub1.count   = 1;
        ub1.stages  = VK_SHADER_STAGE_FRAGMENT_BIT;

        auto& ub2 = layout.addUniformBuffer();
        ub2.set     = 1;
        ub2.binding = 1;
        ub2.size    = sizeof(float) * 8;
        ub2.count   = 1;
        ub2.stages  = VK_SHADER_STAGE_FRAGMENT_BIT;

        layout.finalize();
    }

As you can see, this is all rather straightforward.

Note that the order in which the uniform buffers are added is not relevant, since they are automatically sorted when 
calling :code:`finalize`. Furthermore, when this is done, as well as when exactly the layout is finalized, is entirely 
up to you. The only requirement is that the material is finalized before using it for rendering.

Sharing Method
--------------

By default a separate uniform buffer is allocated for each uniform buffer binding of each material instance. In many 
cases this will be very inefficient. To change this behaviour, you can modify the :code:`sharing.method` and 
:code:`sharing.count` members.

.. code-block:: cpp

    ub0.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Binding;
    ub0.sharing.count  = 8;
    ub1.sharing.method = sol::ForwardMaterialLayout::SharingMethod::InstanceAndBinding;
    ub0.sharing.count  = 16;
    ub2.sharing.method = sol::ForwardMaterialLayout::SharingMethod::InstanceAndBinding;
    ub0.sharing.count  = 16;

Further details can be found on these two pages: :doc:`../../render/forward/forward_material_data` and 
:doc:`../../render/common/uniform_buffer_manager`.

Update Frequency
----------------

.. note::
    Not yet implemented.
