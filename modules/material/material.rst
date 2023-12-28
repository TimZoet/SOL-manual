Material
========

The :code:`sol::Material` class is the base class of all pipeline specific (i.e. compute, graphics and ray tracing)
material classes. It is a pure virtual class and therefore cannot be created directly.

You can add additional functionality to these classes simply by inheriting from them. In fact, this is encouraged.

Descriptor Layouts
------------------

The main shared functionality between the 3 material types, keeping track of descriptor layouts, is implemented in this
class. Note that the material does not own these layouts to make sharing (parts of) the pipelines more efficient.

As an example, consider a common use case where there is a single vertex shader but many different fragment shaders, to
represent various material types. The matrices used in the vertex shader can be put in a single descriptor layout shared
by all materials. Below an excerpt of what the shader code may look like:

.. code-block:: hlsl
    :caption: Snippets of a shared vertex shader and two different fragment shaders.

    // vertex.hlsl

    [[vk::binding(0, 0)]]
    cbuffer Matrices
    {
        float4x4 world;
        float4x4 view;
        float4x4 projection;
    }

    PSInput main(VSInput input) { ... }

    // simple_shading.hlsl

    [[vk::binding(0, 1)]]
    cbuffer SimpleMaterial
    {
        float4 diffuse;
        float4 specular;
        float4 emission;
    }

    float4 main(PSInput input) : SV_TARGET0 { ... }

    // advanced_shading.hlsl

    [[vk::binding(0, 1)]]
    cbuffer AdvancedMaterial
    {
        float4 diffuse;
        float4 specular;
        float3 emission;
        float roughness;
    }
    [[vk::binding(1, 1)]]
    Texture2D<float4> texDiffuse;
    [[vk::binding(2, 1)]]
    Texture2D<float4> texSpecular;
    [[vk::binding(3, 1)]]
    Texture2D<float4> texEmissionRoughness;

    float4 main(PSInput input) : SV_TARGET0 { ... }

Ignoring most of the setup required to define all these pipelines and descriptor layouts, you could have two different
graphics materials whose descriptor layouts are organized as follows:

.. code-block:: cpp
    :caption: Two materials sharing a descriptor layout.

    // Layout with a single uniform buffer binding.
    sol::DescriptorLayoutPtr layoutMatrices = ...;
    // Layout with a single uniform buffer binding.
    sol::DescriptorLayoutPtr   layoutSimple = ...;
    // Layout with a single uniform buffer binding and 3 texture bindings.
    sol::DescriptorLayoutPtr layoutAdvanced = ...;

    // Graphics material that was constructed from a pipeline
    // with shader stages [vertex.hlsl, simple_shading.hlsl]
    // and descriptor layouts [layoutMatrices.get(), layoutSimple.get()].
    sol::GraphicsMaterial& mtlSimple = ...;
    assert(mtlSimple.getDescriptorLayouts()[0] == layoutMatrices.get());
    assert(mtlSimple.getDescriptorLayouts()[1] == layoutSimple.get());

    // Graphics material that was constructed from a pipeline
    // with shader stages [vertex.hlsl, advanced_shading.hlsl]
    // and descriptor layouts [layoutMatrices.get(), layoutAdvanced.get()].
    sol::GraphicsMaterial& mtlAdvanced = ...;
    assert(mtlAdvanced.getDescriptorLayouts()[0] == layoutMatrices.get());
    assert(mtlAdvanced.getDescriptorLayouts()[1] == layoutAdvanced.get());
