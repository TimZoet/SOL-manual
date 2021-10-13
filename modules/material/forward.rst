Forward Material
================

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    forward/instance
    forward/uniform_buffers
    forward/samplers
    forward/push_constants

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

Material Layout
---------------

Naturally, the shaders could contain all sorts of samplers, uniform buffers, push constants, etc. These must all be 
defined through the material layout. After specifying the layout and before a material can be used during rendering, the
layout object must be finalized. It cannot be modified after that.

.. code-block:: cpp

    MyMaterial::MyMaterial(...) : ForwardMaterial(...)
    {
        // Specify material layout.
        auto& ub0 = layout.addUniformBuffer(...);
        ...
        auto& ub1 = layout.addUniformBuffer(...);
        ...

        // Finalize.
        layout.finalize();
    }

Mesh Layout
---------------

Also required is a reference to a :code:`sol::MeshLayout`. Obviously, the vertex shader must match the vertex input 
attributes and bindings as specified in the mesh layout.

.. code-block:: cpp

    // Assuming a previously created mesh layout and material.
    sol::MeshLayout& meshLayout = ...;
    MyMaterial& material        = ...;

    // Assign mesh layout to material.
    material.setMeshLayout(meshLayout);

Material Instance
------------------

The material classes you will be implementing contain just the specification of a material's layout and parameters. 
Actual data, be it textures or uniform buffers, require implementing one or more classes deriving from 
:code:`sol::ForwardMaterialInstance`.
