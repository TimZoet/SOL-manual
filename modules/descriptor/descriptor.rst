Descriptor
==========

The :code:`sol::Descriptor` class owns a region of the internal buffer of a
:doc:`/modules/descriptor/descriptor_buffer`. In this memory, the descriptors for a single
:doc:`/modules/descriptor/descriptor_layout` are stored. Through the descriptor object, these can be easily assigned.

For each type of binding, there is a setter for a single object and a range, e.g. :code:`setSampler` and
:code:`setSamplerRange`. The latter writes a contiguous list of descriptors, which is of course slightly more efficient
than repeatedly calling the single object version. Also, there can be overloads for the types of objects holding the
resource handles. I.e., there is a :code:`setSampler(sol::VulkanSampler,...)`, as well as a
:code:`setSampler(sol::Sampler2D,...)`. The latter of course has a :code:`sol::VulkanSampler` on the inside.

A simple example:

.. code-block:: hlsl
    :caption: A simple descriptor set with two bindings.

    [[vk::binding(0, 0)]]
    Texture2D<float4> color[512];
    [[vk::binding(1, 0)]]
    SamplerState s;

To completely fill the above descriptor, you can do the following:

.. code-block:: cpp
    :caption: Setting up the layout for the above shader.

    // A set of previously created resources.
    sol::Sampler2D& sampler = ...;
    std::vector<sol::Texture2D*> textures = ...;

    // Descriptor for the above shader.
    sol::DescriptorBuffer& buffer = ...;
    sol::DescriptorLayout& layout = ...;
    sol::Descriptor = buffer.allocateDescriptor(layout);

    // From the std::vector assign [0, 128) to descriptor array [0, 128).
    descriptor->setSampledImage(std::span(textures.begin(), 128), 0, 0);
    // From the std::vector assign [1024, 1408) to descriptor array [128, 512).
    descriptor->setSampledImage(std::span(textures.begin() + 1024, 384), 0, 128);

    // Assign a single sampler.
    descriptor->setSampler(sampler, 1, 0);
 
Synchronization is not handled by the descriptor object, but should be handled externally. For example, don't set new
descriptors if they are still in use in a command buffer and place the appropriate barriers and semaphores when doing
transfers using the descriptor buffer.
