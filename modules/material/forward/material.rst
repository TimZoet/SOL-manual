Forward Material
================

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    layout/uniform_buffers
    layout/samplers
    layout/push_constants

A new forward material can be implemented by inheriting from the :code:`sol::ForwardMaterial` class. A forward material
requires a shader module for both the vertex and fragment stage. A possible implementation of a new material class could
more or less look like this:

.. code-block:: cpp

    class MyMaterial : public sol::ForwardMaterial
    {
    public:
        MyMaterial(sol::VulkanShaderModuleSharedPtr vtx,
                   sol::VulkanShaderModuleSharedPtr frag,
                   sol::MeshLayout& mesh) : 
                   ForwardMaterial(std::move(vtx), 
                                   std::move(frag))
        {
            // Specify material layout.
            ...

            // Assign mesh layout.
            setMeshLayout(mesh);
        }
    };

Whether you define a new class for each of your shaders, or deal with this variation inside of a small number of classes
is entirely up to you. The system is very flexible. Also, the pattern used above to hide the constructor and instead use
a static method is just an example to keep the handling of the shader modules within the class.

Material Layout
---------------

Naturally, the shaders could contain all sorts of samplers, uniform buffers, push constants, etc. These must all be 
defined through the material layout. After specifying the layout and before a material can be used during rendering, the
layout object must be finalized. It cannot be modified after that.

.. code-block:: cpp

    MyMaterial::MyMaterial(...) : ForwardMaterial(...)
    {
        // Specify material layout.
        auto& ub   = layout.addUniformBuffer();
        ub.set     = 0;
        ub.binding = 0;
        ...
        auto& sampler   = layout.addSampler();
        sampler.set     = 0;
        sampler.binding = 1;
        ...

        // Finalize material layout. Can no longer be modified.
        layout.finalize();
    }

See the subsections :doc:`layout/uniform_buffers`, :doc:`layout/samplers` and :doc:`layout/push_constants` for detailed
information on how to define each type of data.

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
