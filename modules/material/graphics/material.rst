Graphics Material
=================

The :code:`sol::GraphicsMaterial` class inherits from the :code:`sol::Material` class. It can be used to define and
create graphics pipelines.

Shader Modules
--------------

At present, graphics materials support only vertex and fragment shaders ot be passed on to graphics pipelines.t
Graphics pipelines require only a single shader module. It can be set on construction, or through the
:code:`setGraphicsShader` method. The shader module must be set before creating pipelines from the material. It can be
modified afterwards, though you should be careful with this to prevent confusion when there are still existing pipelines
using the old shader module.

.. code-block:: cpp

    class MyGraphicsMaterial : public sol::GraphicsMaterial
    {
    public:
        MyGraphicsMaterial(sol::ShaderModule& vs, sol::ShaderModule& fs) : GraphicsMaterial(vs, fs)
        {
            ...
        }

        MyGraphicsMaterial(sol::VulkanDevice& device) : GraphicsMaterial(device)
        {
            ...
        }
    };

    // Construct directly with shader modules.
    auto mtl = std::make_unique<MyGraphicsMaterial>(loadShader("/graphics/mymtl.vert.spv"),
                                                    loadShader("/graphics/mymtl.frag.spv"));

    // Or set after construction, but before creating pipelines.
    auto mtl = std::make_unique<MyGraphicsMaterial>(getDevice());
    mtl->setVertexShader(loadShader("/graphics/mymtl.vert.spv"));
    mtl->setFragmentShader(loadShader("/graphics/mymtl.frag.spv"));

Layers
------

.. note::

    TODO.
