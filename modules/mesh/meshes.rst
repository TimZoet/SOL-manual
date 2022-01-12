Meshes
=============

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    meshes/flat_mesh
    meshes/indexed_mesh
    meshes/multi_mesh
    meshes/shared_mesh

There are many different ways in which geometry can be drawn. You can have meshes with just a vertex buffer, or meshes
with many vertex buffers and an index buffer. Either way, these buffers must be bound correctly using 
:code:`vkCmdBindVertexBuffers` and optionally :code:`vkCmdBindIndexBuffer`, after which :code:`vkCmdDraw` or 
:code:`vkCmdDrawIndexed` can be submitted to a command buffer.

The :code:`sol::IMesh` interface defines a number of methods that should return the parameters that are passed to the 
bind and draw commands. A number of classes already implement this interface, and you can of course write your own.
Given a mesh object that implements the interface, it is possible to draw it using the code below:

.. code-block:: cpp

    // Assuming a command buffer to write to and a mesh to draw.
    VkCommandBuffer cmdBuffer = ...;
    sol::IMesh&     mesh      = ...;

    // Skip invalid meshes (vertexCount or indexCount 0, no buffers set, etc...)
    if (!mesh.isValid()) return;

    // Bind the vertex buffers.
    std::vector<VkBuffer> vertexBuffers;
    std::vector<size_t>   vertexBufferOffsets;
    mesh.getVertexBufferHandles(vertexBuffers);
    mesh.getVertexBufferOffsets(vertexBufferOffsets);
    vkCmdBindVertexBuffers(
        cmdBuffer,
        0,
        static_cast<uint32_t>(vertexBuffers.size()),
        vertexBuffers.data(),
        vertexBufferOffsets.data()
    );

    if (!mesh.isIndexed())
    {
        // Draw non-indexed.
        vkCmdDraw(cmdBuffer, mesh.getVertexCount(), 1, mesh.getFirstVertex(), 0);
    }
    else
    {
        // Bind the index buffer.
        vkCmdBindIndexBuffer(
            cmdBuffer,
            mesh.getIndexBufferHandle(),
            mesh.getIndexBufferOffset(),
            mesh.getIndexType()
        );

        // Draw indexed.
        vkCmdDrawIndexed(
            cmdBuffer,
            mesh.getIndexCount(),
            1,
            mesh.getFirstIndex(),
            mesh.getVertexOffset(),
            0
        );
    }

Some notes on all the virtual methods that must be implemented. :code:`isValid` should return :code:`false` if the mesh
cannot be drawn, for example because it has no vertex buffer. None of the getters will be called in that case.

:code:`isIndexed` should return whether the mesh has an index buffer. Again, none of the index buffer related methods
are called if :code:`false` is returned.

The :code:`getVertexBufferHandles` and :code:`getVertexBufferOffsets` append to a :code:`std::vector`. This is mainly to
reduce the number of small allocations that would be made otherwise.

Queue Family Ownership
----------------------

All buffers are created with exclusive ownership (:code:`VK_SHARING_MODE_EXCLUSIVE`). The :code:`sol::MeshManager` takes
care of ownership transfer. On creation, the target queue family is set to the graphics queue, since that is 
where most meshes will be used. Should that not be the case, the target family can be modified using the
:code:`setTargetFamily` method:

.. code-block:: cpp

    // Mesh is going to be used in the compute shader and
    // therefore needs to be owner by the compute queue.
    sol::IMesh& mesh = ...;
    mesh.setTargetFamily(memoryManager->getComputeQueue().getFamily());

Access Flags
----------------------

By default it is assumed that vertex and index buffers are read only as vertex attributes and indices (i.e., the access
flags are VK_ACCESS_VERTEX_ATTRIBUTE_READ_BIT and VK_ACCESS_INDEX_READ_BIT, respectively). This may not always be true,
in which case you must modify the access flags of the buffers. These flags are used during buffer transfer to
synchronize command execution and memory access. Setting the required values can be done with the 
:code:`set(Vertex|Index)AccessFlags` methods provided by the deriving classes:

.. code-block:: cpp

    sol::FlatMesh& mesh = ...;
    mesh.setVertexAccessFlags(VK_ACCESS_SHADER_READ_BIT);

Obviously, you should limit the flags to what is necessary for optimal performance.
