
Buffer To Buffer Copies
=======================

To stage a copy between buffers, call the :code:`stage` method with a filled in
:code:`sol::BufferToBufferCopy`. This struct defines which part of the source buffer to copy to a
specific location in the destination buffer. If you have two buffers of the same size, or if the source buffer is
smaller than the destination buffer, you can leave the value at :code:`VK_WHOLE_SIZE` to automatically copy over the
whole buffer.

To copy all contents of one buffer to another:

.. code-block:: cpp

    // Two equally sized buffers. srcBuffer must have been created
    // with the VK_BUFFER_USAGE_TRANSFER_SRC_BIT usage flag,
    // and dstBuffer with the VK_BUFFER_USAGE_TRANSFER_DST_BIT value.
    sol::IBuffer& srcBuffer = ...;
    sol::IBuffer& dstBuffer = ...;

    auto transaction = manager.beginTransaction();

    // Copy over entirety of srcBuffer to dstBuffer.
    const sol::BufferToBufferCopy copy{
        .srcBuffer              = srcBuffer,
        .dstBuffer              = dstBuffer,
        .size                   = VK_WHOLE_SIZE,
        .srcOffset              = 0,
        .dstOffset              = 0,
        .srcOnDedicatedTransfer = true,
        .dstOnDedicatedTransfer = true,
    };
    transaction->stage(copy);

    transaction->commit();

Explicit Barriers
-----------------

To handle synchronization of the memory, we can do two things. The first is to place a manual 
:doc:`/modules/memory/transaction/buffer_barrier` before and after the copy. The before barrier must take care of
transitioning the buffer from its current state to the transfer destination, while the after barrier goes from the 
transfer state to its final state. Since this is a big hassle, you can also specify explicit barriers as part of the
copy.

Let us pretend that the source buffer is being filled in a compute shader every frame. The destination buffer is being
used as a vertex buffer for a draw command. A separate barrier can be specified for each of these buffers. They need to
specify only the source and destination stage and access, without having to worry about the transfer inbetween:

.. code-block:: cpp

    // Barrier for the source buffer.
    const sol::BufferBarrier srcBarrier{
        .buffer    = srcBuffer,
        // Don't care about destination queue family here.
        .srcFamily = nullptr,
        .dstFamily = nullptr,
        // Wait on compute shader stage.
        .srcStage  = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        // Note no intermediate transfer stage here.
        // That is handled automatically.
        // Next compute shader needs to wait on transfer.
        .dstStage  = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        // Wait on writes.
        .srcAccess = VK_ACCESS_2_SHADER_STORAGE_WRITE_BIT,
        // Note no intermediate transfer access here.
        // That is handled automatically.
        // Next write needs to wait on transfer.
        .dstAccess = VK_ACCESS_2_SHADER_STORAGE_WRITE_BIT
    };

    // Barrier for the destination buffer.
    const sol::BufferBarrier dstBarrier{
        .buffer    = dstBuffer,
        // Don't care about destination queue family here.
        .srcFamily = nullptr,
        .dstFamily = nullptr,
        // Wait on vertex input stage.
        .srcStage  = VK_PIPELINE_STAGE_2_VERTEX_INPUT_BIT,
        // Note no intermediate transfer stage here.
        // That is handled automatically.
        // Next vertex shader needs to wait on transfer.
        .dstStage  = VK_PIPELINE_STAGE_2_VERTEX_INPUT_BIT,
        // Wait on reads.
        .srcAccess = VK_ACCESS_2_VERTEX_ATTRIBUTE_READ_BIT,
        // Note no intermediate transfer access here.
        // That is handled automatically.
        // Next read needs to wait on transfer.
        .dstAccess = VK_ACCESS_2_VERTEX_ATTRIBUTE_READ_BIT
    };

    transaction->stage(copy, srcBarrier, dstBarrier);
