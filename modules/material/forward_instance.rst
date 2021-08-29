Forward Material Instances
==========================

Each forward material class requires a matching forward material instance class. It can be implemented by inheriting 
from the :code:`sol::ForwardMaterialInstance` class. In addition to passing some values to the constructor of the parent
class, there are several virtual methods that must be implemented:

.. code-block:: cpp

    class MyMaterialInstance : public sol::ForwardMaterialInstance
    {
    public:
        MyMaterialInstance(uint32_t identifier,
                           MyMaterial& parentMaterial) : 
                           ForwardMaterialInstance(identifier,
                                                   "MyMaterialInstance", 
                                                   parentMaterial)
            {}

        const void* getUniformBufferData(size_t binding) const noexcept override;
    };


Uniform Buffers
---------------

Consider a forward material with the following fragment shader:

.. code-block:: c

    layout(binding = 0) uniform UBO0
    {
        vec4 value;
    } ubo0;

    layout(binding = 1) uniform UBO1
    {
        vec4 value0;
        vec4 value1;
    } ubo1;

    ...

Reusing the class defined above, we of course need to have member variables representing the data. Simple float arrays
will suffice in this example:

.. code-block:: cpp

    class MyMaterialInstance : public sol::ForwardMaterialInstance
    {
        std::array<float, 4> buffer0;
        std::array<float, 8> buffer1;
        ...
    };

The :code:`getUniformBufferData` method needs to return a pointer to the data belonging to each uniform buffer binding:

.. code-block:: cpp

    const void* MyMaterialInstance::getUniformBufferData(size_t binding) const noexcept
    {
        switch (binding)
        {
            case 0:
                return buffer0.data();
            case 1:
                return buffer1.data();
            default:
                return nullptr;
        }
    }

Samplers
--------

.. note::
    Not yet implemented.
