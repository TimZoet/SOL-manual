Image To Buffer Copy
====================

To stage a copy from an image to a bufer, call the :code:`stage` method with a filled in
:code:`sol::ImageToBufferCopy`. This struct defines which part(s) of the source image to copy to a specific location in
the destination buffer.

.. code-block:: cpp

    // A source image of 512x512 without any mips or layers. Must have
    // been created with the VK_IMAGE_USAGE_TRANSFER_SRC_BIT usage flag.
    // Appropriately sized destination buffer. Must have been created
    // with the VK_BUFFER_USAGE_TRANSFER_DST_BIT value.
    sol::IImage& srcImage = ...;
    sol::IBuffer& dstBuffer = ...;

    auto transaction = manager.beginTransaction();

    // Copy over entirety of srcImage to dstBuffer.
    sol::ImageToBufferCopy copy{
        .srcImage               = srcImage,
        .dstBuffer              = dstBuffer,
        .regions                = {},
        .dstOnDedicatedTransfer = true,
    };
    copy.regions.emplace_back(sol::ImageRegion{
        .dataOffset = 0,
        .aspectMask = VK_IMAGE_ASPECT_COLOR_BIT,
        .mipLevel = 0,
        .baseArrayLayer = 0,
        .layerCount = 1,
        .offset = std::array{0, 0, 0},
        .extent = std::array{512u, 512u, 1u}
    });

    transaction->stage(copy);

    transaction->commit();

Explicit Barriers
-----------------

.. note::

    Not yet implemented.
