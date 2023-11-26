Transaction
===========

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    transaction/buffer_barrier
    transaction/image_barrier
    transaction/staging_buffer
    transaction/staging_image
    transaction/buffer_to_buffer
    transaction/image_to_image
    transaction/buffer_to_image
    transaction/image_to_buffer

The :code:`sol::Transaction` class can be used to stage copies to and between buffers and images. These copies can be
wrapped in barriers to transition the resource to its destination state (both in terms of layout and queue ownership).
Any intermediate barriers to transition to a transfer source/destination are added automatically. Additionally,
standalone barriers, not part of any copy, can be added. The transaction object takes care of recording and submitting
on the right queues.

Multiple transactions can be constructed at the same time. Synchronization between them is handled by the transaction
manager when transactions are committed, i.e. when the command buffers are recorded and submitted. Access to a single
transaction is not thread-safe.

Filling in a transaction is done through the :code:`stage` method. There are several overloads, each of which is
described on its own page. When committed, a single transaction records commands in 3 stages. First, it places the
:code:`BeforeCopy` barriers. After that, any copies between buffers/images or from staging buffers to destination
buffers/images. Finally, the :code:`AfterCopy` barriers. Once recorded, all command buffers are submitted.

.. code-block:: cpp
    :caption: 

    sol::TransactionManager& manager = ...;

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
    :caption: Wait for transaction to complete.

    transaction->commit();
    transaction->wait();
    // Transaction finished, all buffers and images are in their destination state.
    // Also, all intermediate staging buffers are released.

Alternatively, you can retrieve the semaphore (from the transaction manager) and its wait value (from the transaction)
directly. These you can then provide to an arbitrary queue submission elsewhere for a GPU-side wait:

.. code-block:: cpp
    :caption: Using the semaphore handle and state for a submit on the graphics queue.

    transaction->commit();

    // Get semaphore handle and value for graphics queue.
    sol::VulkanQueue&     queue = transaction->getMemoryManager().getGraphicsQueue();
    const uint32_t       family = queue.getFamily().getIndex();
    const VkSemaphore semaphore = transaction->getTransactionManager().getSemaphores()[family]->get();
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
