Buffer Transaction
==================

The :code:`sol::BufferTransaction` class can be used to stage copies to and between buffers. These copies can be wrapped
in memory barriers. The transaction itself takes care of recording and submitting on the right queues.

Multiple transactions can be constructed at the same time. Synchronization between them is handled by the transfer
manager when transactions are committed, i.e. when the command buffers are recorded and submitted. Access to a single
transaction is not thread-safe.

Filling in a transaction is done through the :code:`stage` method. There are several overloads, each of which takes one
of the transaction structs:

* A :code:`sol::BufferTransaction::MemoryBarrier` to be placed before or after the copies.
* A :code:`sol::BufferTransaction::StagingBufferCopy`, which includes a pointer to main memory that is, through a
  staging buffer, copied into a destination buffer.
* A :code:`sol::BufferTransaction::BufferToBufferCopy` defining a transfer between two different buffers.

When committed, a single transaction records commands in 3 stages. First, it places the :code:`BeforeCopy` barriers.
After that, any copies between buffers or from staging buffers to destination buffers. Finally, the :code:`AfterCopy`
barriers. Once recorded, all command buffers are submitted.

.. code-block:: cpp

    sol::TransferManager& manager = ...;

    // Begin a new transaction.
    sol::BufferTransactionPtr transaction = manager.beginTransaction();

    // Stage barriers and/or copies.
    transaction->stage(...);
    ...

    // Record and submit command buffers.
    transaction->commit();

Waiting
-------

After committing, you can safely let the transaction object go out of scope. However, in case you need to wait on the
submitted work to complete, you can do so. To do a CPU-side :code:`vkWaitSemaphores`, call the :code:`wait` method. This
blocks until all semaphores (i.e. the semaphore for each queue that was involved in the transaction) are signaled:

.. code-block:: cpp

    transaction->commit();
    transaction->wait();
    // Transaction finished, all buffers are in their destination state.
    // Also, all intermediate staging buffers are released.

Alternatively, you can retrieve the semaphore (from the transfer manager) and its wait value (from the transaction)
directly. These you can then provide to an arbitrary queue submission elsewhere for a GPU-side wait:

.. code-block:: cpp

    transaction->commit();

    // Get semaphore handle and value for graphics queue.
    sol::VulkanQueue&     queue = transaction->getMemoryManager().getGraphicsQueue();
    const uint32_t       family = queue.getFamily().getIndex();
    const VkSemaphore semaphore = transaction->getTransferManager().getSemaphores()[family]->get();
    const uint64_t        value = transaction->getSemaphoreValues()[family];

    // Fill in semaphore submit.
    const auto wait = VkSemaphoreSubmitInfo{
        .sType       = VK_STRUCTURE_TYPE_SEMAPHORE_SUBMIT_INFO,
        .pNext       = VK_NULL_HANDLE,
        .semaphore   = semaphore,
        .value       = value,
        .stageMask   = VK_PIPELINE_STAGE_2_ALL_COMMANDS_BIT,
        .deviceIndex = 0
    };

    // Fill in submit.
    const VkSubmitInfo2 submit{
        ...
        .waitSemaphoreInfoCount = 1,
        .pWaitSemaphoreInfos    = &wait,
        ...
    };

    // Submit to graphics queue.
    vkQueueSubmit2(queue.get(), 1, &submit, VK_NULL_HANDLE)

Memory Barriers
---------------

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

Staging Buffer Copies
---------------------

To stage a copy from a pointer in main memory to a buffer, call the :code:`stage` method with a filled in
:code:`sol::BufferTransaction::StagingBufferCopy`. This struct holds a pointer to some data that will be placed in a
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
    const sol::BufferTransaction::StagingBufferCopy copy{
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

        const sol::BufferTransaction::StagingBufferCopy copy{
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
        }
    }

    // Commit last transaction.
    transaction->commit();

Note that in the above examples, we are not placing any barriers yet. This means that previous or subsequent usages of
the buffers are not synchronized, and the queue family ownership of the buffers is not yet handled properly. As stated
in the part on memory barriers, you can manually insert barriers before and/or after the call. However, there is also a
slightly more convenient interface which takes a barrier to insert around the copy.

Repeating the example with the storage buffer, let us assume it is going to be read from in a compute shader. Also, we
are going to pretend the buffer already existed and was being used as a uniform in a vertex and fragment shader on the
graphics queue. We can fill in a single :code:`MemoryBarrier` to take care of this:

.. code-block:: cpp

    sol::IBuffer& dstBuffer = ...;
    assert(&dstBuffer.getQueueFamily() == &buffer.getMemoryManager().getGraphicsQueue());

    const sol::BufferTransaction::StagingBufferCopy copy{
        .dstBuffer              = dstBuffer,
        .data                   = data.data(),
        .size                   = data.size(),
        .offset                 = 0,
        .dstOnDedicatedTransfer = true
    };
    
    const sol::BufferTransaction::MemoryBarrier barrier{
        .buffer    = dstBuffer,
        // Move to compute queue.
        .dstFamily = manager.getMemoryManager().getComputeQueue(),
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

One important thing to remember is that staging buffers are not deallocated automatically when you do not explicitly
wait on a transaction. This might mean that subsequent transactions have little to no memory available, until an
explicit wait:

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

        t1->commit();
    }

To help alleviate this, you can pass an additional boolean parameter to the :code:`stage` function to wait on previous
transactions that may be holding the staging buffers:

.. code-block:: cpp

    {
        auto t1 = manager.beginTransaction();
        
        const sol::BufferTransaction::StagingBufferCopy copy{...};
        t1->stage(copy, {}, true);

        t1->commit();
    }

Buffer To Buffer Copies
-----------------------

