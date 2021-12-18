Descriptor Set Layout
=====================

The :code:`sol::VulkanDescriptorSetLayout` class manages the lifetime of a :code:`VkDescriptorSetLayout`. To create one,
bindings must be specified that match what's in the shaders:

.. code-block:: cpp

    // Vertex shader:

    layout(set = 0, binding = 0) uniform SomeUniformBuffer
    {
        ...
    } ub;

    // Fragment shader:

    layout(set = 0, binding = 1) uniform sampler2D tex;

    // C++:

    VulkanDescriptorSetLayout::Settings settings;
    settings.device      = device
    settings.bindings.emplace_back(VkDescriptorSetLayoutBinding{
        .binding            = 0,
        .descriptorType     = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER,
        .descriptorCount    = 1,
        .stageFlags         = VK_SHADER_STAGE_VERTEX_BIT,
        .pImmutableSamplers = nullptr
    });
    settings.bindings.emplace_back(VkDescriptorSetLayoutBinding{
        .binding            = 1,
        .descriptorType     = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER,
        .descriptorCount    = 1,
        .stageFlags         = VK_SHADER_STAGE_FRAGMENT_BIT,
        .pImmutableSamplers = nullptr
    });

    auto layout = sol::VulkanDescriptorSetLayout::create(settings);
