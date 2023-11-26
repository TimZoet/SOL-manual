Staging Buffer Copy
===================

To stage a copy from a pointer in main memory to a buffer, call the :code:`stage` method with a filled in
:code:`sol::StagingBufferCopy`. This struct holds a pointer to some data that will be placed in a
staging buffer:

.. code-block:: cpp

    // Let's assume we have some data to place in a storage buffer.
    std::vector<uint8_t> data = ...;

    // Create storage buffer that is also a transfer destination.
    const sol::IBufferAllocator::AllocationInfo info{
        .size        = data.size(),
        .bufferUsage = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT |
                       VK_BUFFER_USAGE_TRANSFER_DST_BIT,
        ...
    };
    auto dstBuffer = manager.getMemoryManager().allocateBuffer(
        info,
        sol::IBufferAllocator::OnAllocationFailure::Throw
    )

    auto transaction = manager.beginTransaction();
    
    // This will immediately copy the data to an intermediate staging buffer...
    const sol::StagingBufferCopy copy{
        .dstBuffer              = *dstBuffer,
        .data                   = data.data(),
        .size                   = data.size(),
        .offset                 = 0,
        .dstOnDedicatedTransfer = true
    };
    transaction->stage(copy);

    // ...so the data pointer we passed can be freed directly after the call.
    data.clear();

    // Committing will submit a copy from the staging to the destination buffer.
    transaction->commit();

Allocation Failure
------------------

It could be that the memory pool from which staging buffers are being allocated is already full because another
transaction is ongoing or you are attempting to copy more data in the current transaction than fits in the pool. To that
end, you should always check the return value of the :code:`stage` call. If it returns false, there was an allocation
failure. You can commit any transactions that are still holding staging memory and, if that is not enough, copy the
remaining data through a separate commit:

.. code-block:: cpp

    auto transaction = manager.beginTransaction();

    while (dataRemaining())
    {
        const std::vector<uint8_t> data = retrieveNextData();
        sol::IBuffer&         dstBuffer = retrieveNextBuffer();

        const sol::StagingBufferCopy copy{
            .dstBuffer              = dstBuffer,
            .data                   = data.data(),
            .size                   = data.size(),
            .offset                 = 0,
            .dstOnDedicatedTransfer = true
        };

        // If memory pool is full, commit transaction and make a new one
        // to redo the current and remaining copies.
        if (!transaction->stage(copy))
        {
            transaction->commit();
            transaction = getTransferManager().beginTransaction();
            resetNextData(data);
            resetNextBuffer(dstBuffer);
        }
    }

    // Commit last transaction.
    transaction->commit();

One important thing to remember is that staging buffers are not deallocated automatically when you do not explicitly
wait on a transaction, even when the transaction object is destroyed.  This might mean that subsequent transactions have
little to no memory available, until an explicit wait:

.. code-block:: cpp

    {
        auto t0 = manager.beginTransaction();
        
        // Stage some copies involving staging buffers.
        ...
        
        t0->commit();
    } // We did not explicitly wait, so staging buffers for t0 are still allocated.

    {
        auto t1 = manager.beginTransaction();
        
        // We might run out of staging buffer memory unexpectedly here.
        ...

        // Though t0's staging buffers are cleared during t1's commit.
        t1->commit();
    }

To help alleviate this, you can pass an additional boolean parameter to the :code:`stage` function to wait on previous
transactions that may be holding the staging buffers:

.. code-block:: cpp

    {
        auto t1 = manager.beginTransaction();
        
        const sol::StagingBufferCopy copy{...};
        t1->stage(copy, {}, true);

        t1->commit();
    }

Explicit Barriers
-----------------

Note that in the above examples, we are not placing any barriers yet. This means that previous or subsequent usages of
the buffers are not synchronized, and the queue family ownership of the buffers is not yet handled properly. As stated
in :doc:`/modules/memory/transaction/buffer_barrier`, you can manually insert barriers before and/or after the call.
However, there is also a slightly more convenient interface which takes an explicit barrier to insert around the copy.

Repeating the example with the storage buffer, let us assume it is going to be read from in a compute shader. Also, we
are going to pretend the buffer already existed and was being used as a uniform in a vertex and fragment shader on the
graphics queue. We can fill in a single :code:`sol::BufferBarrier` to take care of this:

.. code-block:: cpp

    sol::IBuffer& dstBuffer = ...;
    assert(&dstBuffer.getQueueFamily() == &buffer.getMemoryManager().getGraphicsQueue());

    const sol::StagingBufferCopy copy{
        .dstBuffer              = dstBuffer,
        .data                   = data.data(),
        .size                   = data.size(),
        .offset                 = 0,
        .dstOnDedicatedTransfer = true
    };
    
    const sol::BufferBarrier barrier{
        .buffer    = dstBuffer,
        // Move to compute queue.
        .dstFamily = &manager.getMemoryManager().getComputeQueue(),
        // Wait on vertex and fragment shader stage.
        .srcStage  = VK_PIPELINE_STAGE_2_VERTEX_SHADER_BIT | VK_PIPELINE_STAGE_2_FRAGMENT_SHADER_BIT,
        // Compute shader stage needs to wait.
        .dstStage  = VK_PIPELINE_STAGE_2_COMPUTE_BIT,
        // Wait on uniform reads.
        .srcAccess = VK_ACCESS_2_UNIFORM_READ_BIT,
        // Storage reads need to wait.
        .dstAccess = VK_ACCESS_2_SHADER_STORAGE_READ_BIT
    };

    transaction->stage(copy, barrier);

Note that you do not have to deal with anything happening on the transfer queue. Ownership is transfered automatically,
and the appropriate transfer stage and access flags are placed.
