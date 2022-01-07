Uniform Buffer Manager
======================

The :code:`sol::UniformBufferManager` class handles the allocation and sharing of uniform buffers. It is agnostic to
the type of materials that are used, be they forward, deferred, or anything else. The built-in material managers make
use of this class to optimize their usage of uniform buffers when dealing with many material instances. Should you be 
implementing your own material manager or something similar, you could also make use of this class.

Construction of a :code:`sol::UniformBufferManager` requires a reference to a :code:`sol::MemoryManager` and the number 
of separate/duplicate buffers that must be allocated with each :code:`sol::UniformBuffer` instance. This last parameter 
is important when you are e.g. using double (or triple, or...) buffering to render multiple frames at once.

.. code-block:: cpp

    // Assuming a previously created MemoryManager.
    sol::MemoryManager memoryManager = ...;

    // Create a new UniformBufferManager. Pass in 2 to get double buffering.
    sol::UniformBufferManager ubManager(memoryManager, 2);

The below diagram displays the relationship between the classes described here. In this image, :code:`B` is the number 
of duplicate buffers that is allocated, while :code:`N` corresponds to the maximum number of slots each uniform buffer 
can fit.

.. image:: /_static/images/material/uniform_buffer_manager.svg
    :alt: Diagram of the UniformBufferManager.
    :align: center

Uniform Buffers
---------------

When requesting a new uniform buffer, you must specify for which material, set and binding it is used. That way, the 
manager can look for already existing uniform buffers that are not yet full. When storing multiple bindings in a single
uniform buffer (more on that below) you must provide a unique binding. Using the first binding is the most practical and
would already suffice.

Also needed are the sharing method, the number of slots, and the size required for a single slot. The sharing method 
determines how buffers are shared between bindings and material instances. The slot size is the size of the binding in 
bytes. In the case of multiple bindings it should be the sum of their sizes. A detailed explanation will follow below.
Which method is most optimal will of course differ for each use case. You'll have to profile this yourself.

As an example, what follows is a (massively simplified) excerpt from what happens in the forward material manager. In
this example, we only want to share uniform buffers for the same bindings:

.. code-block:: cpp

    // Assuming some ForwardMaterialInstance.
    sol::ForwardMaterialInstancePtr instance = ...;
    const auto& material                     = instance->getForwardMaterial();

    // Iterate over all sol::UniformBufferBindings belonging to the current material instance.
    for (const auto* ubb : material.getLayout().getUniformBuffers(setIndex))
    {
        // Create a uniform buffer that will share the data of this binding between 4 material instances.
        auto* uniformBuffer = ubManager.getOrCreate(
            &material, 
            instance->getSetIndex(), 
            ubb->binding, 
            sol::ForwardMaterialLayout::SharingMethod::Binding,
            4,
            ubb->size * ubb->count
        );

        // Get a free slot in the uniform buffer and store it somewhere so that 
        // the data can be updated when the material instance is modified, 
        // for retrieval during command buffer recording,
        // or deletion once the material instance is no longer in use.
        const auto [slot, offset] = uniformBuffer->getFreeSlot();
        ...
    }

The first time the above code runs (i.e. the first time a new instance of a particular forward material is created), a 
new uniform buffer is made, together with however many :code:`sol::VulkanBuffers` are needed for double buffering. For
the next 3 material instances, :code:`getOrCreate` will return the same uniform buffers.

Writing Data
------------

When to write new data is up to the higher layers. The manager or uniform buffers themselves do not know where the data
comes from and when the memory in the buffers has become stale.

In order to write data to the allocated buffers, there are various methods to :code:`map`, :code:`unmap` and 
:code:`flush` them. Currently, all buffers are :code:`HOST_VISIBLE` such that they can be mapped. In the future, new 
options may be introduced to have buffers be only visible to the GPU. This could mainly be useful for infrequently 
updated data where staging buffers would give better performance.

Sharing Method
--------------

In order to explain the various sharing methods a bit better, we will be looking at how uniform buffers are created for 
forward materials. Consider the definition of a forward material layout below:

.. code-block:: cpp

    auto& ub0   = layout.addUniformBuffer();
    ub0.set     = 0;
    ub0.binding = 0;
    ub0.size    = 32;

    auto& ub1   = layout.addUniformBuffer();
    ub1.set     = 0;
    ub1.binding = 1;
    ub1.size    = 16;

    auto& ub2   = layout.addUniformBuffer();
    ub2.set     = 1;
    ub2.binding = 0;
    ub2.size    = 48;

    auto& ub3   = layout.addUniformBuffer();
    ub3.set     = 1;
    ub3.binding = 1;
    ub3.size    = 32;

This is perhaps better visualized using the next image. On the right a snippet of what the layout definition in the 
shader code could be. On the left a more visual representation:

.. image:: /_static/images/material/shader_layout.svg
    :alt: Visual representation of the layout defined in the previous code snippet.
    :align: center

By default, none of the bindings will be shared. Each time a new material instance is created, separate buffers are 
allocated. If we create a material instance for both descriptor set 0 and 1, we would get two buffers for each instance:

.. image:: /_static/images/material/shared_none.svg
    :alt: Diagram of buffer layout when no bindings are shared.
    :align: center

Alternatively, it is possible to share the bindings within the same descriptor set, or material instance. This can be 
specified in the material layout through the :code:`sharing.method` member:

.. code-block:: cpp

    ub0.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Instance;
    ub1.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Instance;
    ub2.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Instance;
    ub3.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Instance;

Whenever a new material instance is created, a single buffer is allocated. Again visualized for a single instance for 
descriptor set 0 and 1:

.. image:: /_static/images/material/shared_instance.svg
    :alt: Diagram of buffer layout when bindings within the same material instance are shared.
    :align: center

Another option is to have the same binding of multiple material instances share a buffer. In that case, you must also
define how many instances will be placed in a single buffer. This can be done per binding:

.. code-block:: cpp

    ub0.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Binding;
    ub0.sharing.count  = 4;
    ub1.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Binding;
    ub1.sharing.count  = 4;
    ub2.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Binding;
    ub2.sharing.count  = 2;
    ub3.sharing.method = sol::ForwardMaterialLayout::SharingMethod::Binding;
    ub3.sharing.count  = 4;

If we now create a new material instance for descriptor set 0, two new buffers are allocated, one for each binding. 
However, for the next three instances, the same buffers would get reused. For set 1, things are a bit different. A new 
buffer for binding 0 is already allocated after two instances, while the binding 1 buffer still fits four instances. 
Four material instances would therefore result in a total of three buffers:

.. image:: /_static/images/material/shared_binding.svg
    :alt: Diagram of buffer layout when the same binding of multiple material instance is shared.
    :align: center

Finally, multiple bindings of multiple material instances can be shared by changing the sharing method to 
:code:`InstanceAndBinding`. If the :code:`sharing.count` is different between the bindings, the count of the first 
binding of the set is used. In this example, set 0 will use a count of 4, and set 1 a count of 2:

.. image:: /_static/images/material/shared_both.svg
    :alt: Diagram of buffer layout when bindings of multiple material instances are shared.
    :align: center

Note that it is possible to use a different sharing method for bindings within the same set. You could have one binding 
being shared between material instances, while the remaining bindings are shared for just a single instance. Mind you, 
for some material layouts, different sharing configurations can be effectively identical. Finally, it is not allowed to 
create uniform buffers for overlapping sets of bindings.

Update Frequency
----------------

.. note::
    Not yet implemented.

Deallocation
------------

.. note::
    Not yet implemented.
