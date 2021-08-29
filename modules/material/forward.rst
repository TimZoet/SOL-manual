Forward Materials
=================

A new forward material can be implemented by inheriting from the :code:`sol::ForwardMaterial` class. A forward material
requires a shader module for both the vertex and fragment stage. A possible implementation of a new material class could
more or less look like this:

.. code-block:: cpp

    class MyMaterial : public sol::ForwardMaterial
    {
        MyMaterial(sol::VulkanShaderModuleSharedPtr vtx,
                   sol::VulkanShaderModuleSharedPtr frag) : 
                   ForwardMaterial("MyMaterial", 
                                   std::move(vtx), 
                                   std::move(frag))
        {
            // Specify material layout.
            ...
        }
    
    public:
        static std::unique_ptr<MyMaterial> create(...)
        {
            // Retrieve vertex and fragment shader module from somewhere.
            sol::VulkanShaderModuleSharedPtr vtx = ...;
            sol::VulkanShaderModuleSharedPtr frag = ...;

            // Create material.
            return std::make_unique<MyMaterial>(std::move(vtx), std::move(frag));
        }
    };

Whether you define a new class for each of your shaders, or deal with this variation inside of a small number of classes
is entirely up to you. The system is very flexible.

Naturally, the shaders could contain all sorts of samplers, uniform buffers, etc. These must all be defined through the
material layout. After specifying the layout and before a material can be used during rendering, the layout object must 
be finalized. Materials cannot be modified after that.

.. figure:: /_static/images/forward_material_layout.svg
    :alt: Diagram of the forward material layout.

Uniform Buffers
---------------

Consider a forward material with the following shaders:

.. code-block:: c

    // Vertex shader:

    layout(binding = 1) uniform UBO1
    {
        mat4 value;
    } ubo1[3];

    ...

    // Fragment shader:

    layout(binding = 0) uniform UBO0
    {
        vec4 value;
    } ubo0;

    layout(binding = 2) uniform UBO2
    {
        vec4 value0;
        vec4 value1;
    } ubo2;

    ...

This material's layout can be defined as follows:

.. code-block:: cpp

    MyMaterial::MyMaterial(...)
    {
        layout.addUniformBuffer(0, sizeof(float) * 4, 1, VK_SHADER_STAGE_FRAGMENT_BIT);
        layout.addUniformBuffer(1, sizeof(float) * 16, 3, VK_SHADER_STAGE_VERTEX_BIT);
        layout.addUniformBuffer(2, sizeof(float) * 4 * 2, 1, VK_SHADER_STAGE_FRAGMENT_BIT);
        layout.finalize();
    }

Samplers
--------

.. note::
    Not yet implemented.
