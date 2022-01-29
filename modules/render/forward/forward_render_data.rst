Forward Render Data
===================

The :code:`sol::ForwardRenderData` class defines the layout of the data that can be passed to the
:code:`sol::ForwardRenderer`. It contains a list of :code:`Drawables`, each of which has all the data needed to perform
a draw call. See the below diagram.

.. figure:: /_static/images/render/forward/forward_render_data.svg
    :alt: Diagram of the material class hierarchy.

For a draw call, the following 4 things are needed:

#. A graphics pipeline.
#. Index and vertex buffers.
#. A list of descriptor sets.
#. Push constant data.

A drawable has a single reference to a :code:`sol::ForwardMaterial`. This material should be used to construct (or
retrieve) the graphics pipeline to be bound using :code:`vkCmdBindPipeline`.

It also has a single reference to a :code:`sol::IMesh`. This mesh contains the index and vertex buffers to be bound
using :code:`vkCmdBindIndexBuffer` and :code:`vkCmdBindVertexBuffers`.

The material defines a number of descriptor sets in its layout. For each set, a material instance is needed. The
drawable has a single offset into the :code:`materialInstances` list of the render data. Starting at this offset,
there is a reference to a :code:`sol::ForwardMaterialInstance` for each descriptor set. These material instances are
guaranteed to be sorted by set index. They can be used to to obtain the parameters of :code:`vkCmdBindDescriptorSets`.

Similar to the descriptor sets, a material layout also defines a list of push constant ranges. The drawable again has a
single offset, this time into the :code:`pushConstantoffsets` list of the render data. For each push constant range in
the material layout, there is an offset (in bytes) into the :code:`pushConstantData` data array.
