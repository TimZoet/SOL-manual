Material Instance
=================

The :code:`sol::MaterialInstance` class is the base class of all pipeline type specific (i.e. compute, graphics and ray
tracing) material instance classes. It is a pure virtual class and therefore cannot be created directly.

You can add additional functionality to these classes simply by inheriting from them. In fact, this is encouraged.

Descriptors
-----------

Material instances hold one or more :doc:`/modules/descriptor/descriptor` objects, one for each descriptor layout in the
material. However, not all descriptors need to be enabled. A material instance can have only a subset of descriptors
enabled. This can be used for more efficient sharing of descriptor data. Consider an object that consists of multiple
submeshes requiring different draw calls because each mesh uses different textures. The transformation matrix for each
submesh may still be the same. Assuming the bindings are spread over two different descriptor layouts, you could have
one material instance with the descriptor for the object's matrix, and one material instance for each submesh with the
other descriptor containing textures.

When an instance is created, no descriptors are enabled by default. You can do so using the :code:`enableDescriptor`
method, which takes either an index or a reference to a descriptor layout. If the descriptor at the given index was not
yet enabled, a new descriptor is allocated from the provided :doc:`/modules/descriptor/descriptor_buffer`:

.. code-block:: cpp
    :caption: Enabling descriptors on a material instance.

    // Assuming a previously created material with 2 descriptor layouts,
    // and a buffer to allocate descriptors from.
    sol::GraphicsMaterial&    mtl = ...;
    sol::DescriptorBuffer& buffer = ...;

    auto mtlInstance = mtl.createInstance<sol::GraphicsMaterialInstance>();

    // Enable first descriptor by index and second by reference to layout.
    auto& desc0 = mtlInstance->enableDescriptor(0, buffer);
    auto& desc1 = mtlInstance->enableDescriptor(mtl.getDescriptorLayouts()[1], buffer);

    // Fill the descriptors.
    desc0.setUniformBuffer(...);
    desc1.setSampledImage(...);
    desc1.setSampledImage(...);

Obviously, you can also disable a descriptor again using :code:`disableDescriptor`. This will deallocate the descriptor
object.

To streamline working with material instances, you can iterate over them to get the list of enabled descriptors:

.. code-block:: cpp

    sol::MaterialInstance& mtlInstance = ...;
    mtlInstance.enableDescriptor(0, buffer);
    mtlInstance.enableDescriptor(2, buffer);

    // Assuming descriptor[1] was never enabled, this should give descriptor 0 and 2:
    for (auto [descriptor, index] : mtlInstance)
    { ... }
