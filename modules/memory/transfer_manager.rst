Transfer Manager
================

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    transfer/buffer_transaction

The :code:`sol::TransferManager` class coordinates buffer and image transactions. Transactions record barriers, copies
between buffers/images, and copies through a staging buffer. Creation requires only a memory manager and the size of the
:doc:`/modules/memory/pools/ring` that is used to allocate staging buffers:

.. code-block:: cpp

    sol::MemoryManager& memoryManager = ...;

    // Create a transfer manager with a 256MiB ring buffer memory pool.
    auto transferManager = sol::TransferManager::create(
        memoryManager,
        256ull * 1024ull * 1024ull
    );
