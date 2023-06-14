Material Layout
===============

The :code:`sol::MaterialLayout` class turns a :code:`sol::MaterialLayoutDescription` into a collection of
:code:`VkDescriptorSetLayouts` and :code:`VkPushConstantRanges`. Deriving classes can add additional behaviour. For
example, the :code:`sol::GraphicsMaterialLayout` adds options to define pipeline dynamic states.

Creation of the descriptor set layouts and push constant ranges is done in the :code:`finalize` method:

.. code-block:: cpp

    // Setup a layout description.
    sol::MaterialLayoutDescription desc;
    ...

    // Use description to finalize a sol::ComputeMaterialLayout.
    sol::ComputeMaterialLayout& layout = ...;
    layout.finalize(desc);

After finalization, a large number of methods can be used to inspect the material layout and retrieve the created Vulkan
objects. With that information, pipelines and descriptor sets can be constructed.

