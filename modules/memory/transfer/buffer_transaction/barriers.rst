Memory Barriers
===============

To place a memory barrier that optionally transfers queue ownership, call the :code:`stage` method with a filled in
:code:`sol::BufferTransaction::MemoryBarrier`. This also requires specifying the :code:`BarrierLocation` which
determines whether the barrier is placed before or after any copies. This is only relevant when doing copies on the
buffer in the same transaction.

As a simple example, imagine we want to transfer ownership of a buffer from the graphics queue to the compute queue. It
is being written to in a vertex shader and a compute shader will read the results. As long as no copies are done on the
buffer at the same time, we don't really care at what location the transaction places the barrier, exactly:

.. code-block:: cpp

    // Buffer that was being written to in a shader on the graphics queue. 
    sol::IBuffer& buffer = ...;
    assert(&buffer.getQueueFamily() == &buffer.getMemoryManager().getGraphicsQueue());

    auto transaction = manager.beginTransaction();

    // Stage ownership transfer to compute queue with appropriate stage and access masks.
    // We place the barrier before any copies, though it doesn't really matter in this case.
    const sol::BufferTransaction::MemoryBarrier barrier{
        .buffer    = buffer,
        .dstFamily = &buffer.getMemoryManager().getComputeQueue(),
        .srcStage  = VK_PIPELINE_STAGE_2_VERTEX_SHADER_BIT,
        .dstStage  = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        .srcAccess = VK_ACCESS_2_SHADER_STORAGE_WRITE_BIT,
        .dstAccess = VK_ACCESS_2_SHADER_STORAGE_READ_BIT
    };
    transaction->stage(barrier, sol::BufferTransaction::BarrierLocation::BeforeCopy);
    transaction->commit();

    // After the commit, the queue family of the buffer was updated on the CPU side.
    assert(&buffer.getQueueFamily() == &buffer.getMemoryManager().getComputeQueue());

    // Here, we can safely submit some work on the compute queue that reads from the buffer.
    ...

If you are doing copies at the same time, you can of course manually place the required barriers. However, this is error
prone and will involve a lot of typing. When staging copies, barriers can be specified through optional parameters. See
also the next sections.
