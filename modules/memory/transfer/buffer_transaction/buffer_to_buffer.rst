
Buffer To Buffer Copies
=======================

To stage a copy between buffers, call the :code:`stage` method with a filled in
:code:`sol::BufferTransaction::BufferToBufferCopy`. This struct defines which part of the source buffer to copy to a
specific location in the destination buffer. If you have two buffers of the same size, b

To copy all contents of one buffer to another:

.. code-block:: cpp

    // Two equally sized buffers. srcBuffer must have been created
    // with the VK_BUFFER_USAGE_TRANSFER_SRC_BIT usage flag,
    // and dstBuffer with the VK_BUFFER_USAGE_TRANSFER_DST_BIT value.
    sol::IBuffer& srcBuffer = ...;
    sol::IBuffer& dstBuffer = ...;

    auto transaction = manager.beginTransaction();

    // Copy over entirety of srcBuffer to dstBuffer.
    const sol::BufferTransaction::BufferToBufferCopy copy{
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

As with the staging buffer copies, we must either place manual barriers before and after the copy, or specify them as
part of the copy. Let us pretend that the source buffer is being filled in a compute shader every frame. The destination
buffer is being used as a vertex buffer for a draw command. A separate barrier can be specified for the source and
destination buffer:

.. code-block:: cpp

    const sol::BufferTransaction::MemoryBarrier srcBarrier{
        .buffer    = srcBuffer,
        .dstFamily = nullptr,
        // Wait on compute shader stage.
        .srcStage  = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        // Next compute shader needs to wait on transfer.
        .dstStage  = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        // Wait on writes.
        .srcAccess = VK_ACCESS_2_SHADER_STORAGE_WRITE_BIT,
        // Next write needs to wait on transfer.
        .dstAccess = VK_ACCESS_2_SHADER_STORAGE_WRITE_BIT
    };

    const sol::BufferTransaction::MemoryBarrier dstBarrier{
        .buffer    = dstBuffer,
        .dstFamily = nullptr,
        .srcStage  = VK_PIPELINE_STAGE_2_VERTEX_INPUT_BIT,
        .dstStage  = VK_PIPELINE_STAGE_2_VERTEX_INPUT_BIT,
        .srcAccess = VK_ACCESS_2_VERTEX_ATTRIBUTE_READ_BIT,
        .dstAccess = VK_ACCESS_2_VERTEX_ATTRIBUTE_READ_BIT
    };

    transaction->stage(copy, srcBarrier, dstBarrier);
