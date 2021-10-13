Forward Material Instance
=========================

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
