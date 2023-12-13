Index Buffers
=============

The :code:`sol::IndexBuffer` class manages a index buffer created through the
:doc:`/modules/mesh/geometry_buffer_allocator`. This buffer can be a suballocation of a global buffer, depending on how
the allocator was configured.

Because this class inherits from :code:`sol::IBuffer`, the normal methods for copying data and placing barriers are
available. However, there are also dedicated :code:`getIndexData` and :code:`setIndexData` that deal with counts of
indices instead of bytes.

.. code-block:: cpp
    :caption: Transferring data to an index buffer.
    
    sol::TransactionManager&        manager = ...;
    sol::GeometryBufferAllocator& allocator = ...;
    
    // We have some data that we want to put into an index buffer.
    const std::vector<uint16_t>   data = ...;
    sol::IndexBufferPtr        buffer = allocator.allocateIndexBuffer(data.size(), sizeof(uint16_t));

    const auto transaction = manager.beginTransaction();

    // Because allocation of the staging buffer might fail,
    // you should always check the return value.
    const auto success = buffer->setIndexData(
        *transaction,
        data.data(),
        data.size(),
        0,
        sol::IBuffer::Barrier{
            .dstFamily = &manager.getMemoryManager().getGraphicsQueue().getFamily(),
            // There were no previous commands yet.
            .srcStage  = VK_PIPELINE_STAGE_2_NONE,
            // Buffer is going to be used as input for draw commands.
            .dstStage  = VK_PIPELINE_STAGE_2_INDEX_INPUT_BIT,
            // There were no previous commands yet.
            .srcAccess = VK_ACCESS_2_NONE,
            // Buffer is going to be used as input for draw commands.
            .dstAccess = VK_ACCESS_2_INDEX_READ_BIT
        },
        false
    );

    if (!success)
        throw std::runtime_error("Staging buffer allocation failed.");

    // Commit to perform the copy.
    transaction->commit();
    transaction->wait();

    // We can now safely use the buffer.
