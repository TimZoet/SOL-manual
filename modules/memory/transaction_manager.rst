Transaction Manager
===================

The :code:`sol::TransactionManager` class coordinates buffer and image transactions. Transactions record barriers,
copies between buffers/images, and copies through a staging buffer. Creation requires only a memory manager and the size
of the :doc:`/modules/memory/pools/ring` that is used to allocate staging buffers:

.. code-block:: cpp
    :caption: Creating a transaction manager.

    sol::MemoryManager& memoryManager = ...;

    // Create a transaction manager with a 256MiB ring buffer memory pool.
    auto transactionManager = sol::TransactionManager::create(
        memoryManager,
        256ull * 1024ull * 1024ull
    );

To create a new transaction, call the manager's :code:`beginTransaction` method. Filling and committing the transaction
is done through the :doc:`/modules/memory/transaction` object itself:

.. code-block:: cpp
    :caption: Doing a transaction.

    // Begin a new transaction.
    sol::BufferTransactionPtr transaction = transactionManager->beginTransaction();

    // Stage barriers and/or copies.
    transaction->stage(...);
    ...

    // Record and submit command buffers.
    transaction->commit();
