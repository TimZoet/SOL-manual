Descriptor Layout
=================

.. note::

    There are no ways yet to automate the creation of layouts through reflection.

The :code:`sol::DescriptorLayout` class describes the bindings of a single descriptor set. With a finalized layout, you
can allocate :doc:`/modules/descriptor/descriptor` objects from a :doc:`/modules/descriptor/descriptor_buffer`.

Creating layouts is fairly straightforward. There are a number of structures that can be filled in to describe a binding
and passed to the layout using the :code:`add` method. As an example, consider the following shader:

.. code-block:: hlsl
    :caption: A pixel shader with 4 bindings in 2 descriptor sets.

    struct PSInput
    {
        float2 uv : TEXCOORD0;
    };

    struct Light
    {
        float3 intensity;
        float range;
        float3 position;
        uint layer;
        float3 direction;
        float dummy;
    };

    struct Material
    {
        uint albedo;
        uint emission;
    };

    struct Instance
    {
        uint index;
    };

    // Descriptor 0.
    [[vk::binding(0, 0)]]
    cbuffer ColorCorrection
    {
        float4 coeff_a;
        float4 coeff_b;
        float4 coeff_c;
    };
    [[vk::binding(1, 0)]]
    StructuredBuffer<Material> materials;

    // Descriptor 1.
    [[vk::binding(0, 1)]]
    Texture2D<float4> color[512];
    [[vk::binding(1, 1)]]
    SamplerState s;

    [[vk::push_constant]]
    ConstantBuffer<Instance> instance;

    float4 main(PSInput input) : SV_TARGET0
    {
        Material material = materials[instance.index];
        float4 x = color[material.albedo].Sample(s, input.uv);
        return coeff_a * x * x + coeff_b * x + coeff_c;
    }

The two descriptor sets used in the shader can be created as follows:

.. code-block:: cpp
    :caption: Setting up the layouts for the above shader.

    sol::VulkanDevice& device = ...;

    // Create layout for descriptor 0.
    auto layout0 = std::make_unique<sol::DescriptorLayout>(device);
    layout0->add(sol::DescriptorLayout::UniformBufferBinding{
        .binding = 0,
        .size = sizeof(float4) * 3,
        .count = 1,
        .stages = VK_SHADER_STAGE_FRAGMENT
    });
    layout0->add(sol::DescriptorLayout::StorageBufferBinding{
        .binding = 1,
        .count = 1,
        .stages = VK_SHADER_STAGE_FRAGMENT
    };
    layout0->finalize();

    // Create layout for descriptor 1.
    auto layout1 = std::make_unique<sol::DescriptorLayout>(device);
    layout1->add(sol::DescriptorLayout::SampledImageBinding{
        .binding = 0,
        .count = 512,
        .stages = VK_SHADER_STAGE_FRAGMENT
    };
    layout1->add(sol::DescriptorLayout::SamplerBinding{
        .binding = 1,
        .count = 1,
        .stages = VK_SHADER_STAGE_FRAGMENT
    };
    layout1->finalize();
