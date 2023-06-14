Compute Material
================

The :code:`sol::ComputeMaterial` class inherits from the :code:`sol::Material` class. It can be used to define and
create compute pipelines.

Shader Modules
--------------

Compute pipelines require only a single shader module. A compute material can be assigned one on construction, or
through the :code:`setComputeShader` method. The shader module must be set before creating pipelines from the material.
It can be modified afterwards, though you should be careful with this to prevent confusion when there are still existing
pipelines using the old shader module.

.. code-block:: cpp

    class MyComputeMaterial : public sol::ComputeMaterial
    {
    public:
        MyComputeMaterial(sol::ShaderModule& cs) : ComputeMaterial(cs)
        {
            ...
        }

        MyComputeMaterial(sol::VulkanDevice& device) : ComputeMaterial(device)
        {
            ...
        }
    };

    // Construct directly with shader module.
    auto mtl = std::make_unique<MyComputeMaterial>(loadShader("/compute/mycompute.cs.spv"));

    // Or set after construction, but before creating pipelines.
    auto mtl = std::make_unique<MyComputeMaterial>(getDevice());
    mtl->setComputeShader(loadShader("/compute/mycompute.cs.spv"));
