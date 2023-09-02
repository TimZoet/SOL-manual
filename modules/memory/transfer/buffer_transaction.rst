Buffer Transaction
==================

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    buffer_transaction/barriers
    buffer_transaction/staging_buffer
    buffer_transaction/buffer_to_buffer

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
