Images
======

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    images/image1d
    images/image2d
    images/image3d
    images/image1darray
    images/image2darray
    images/image3darray

The :code:`sol::Image...` classes manage various types of images: color and depth images, image arrays, with and without
mipmaps, etc.

.. code-block:: cpp
    :caption: Creating and destroying some images.

    sol::TextureCollection& collection = ...;

    // Create a number of images.
    auto& imageA = collection.createImage2D({1024, 1024}, ...);
    auto& imageB = collection.createImage3D({1024, 512}, ...);
    auto& imageC = collection.createImage2DArray({1024, 512}, ...);

    // Do some stuff with images, like uploading data and making textures.
    ...

    // Deallocate once they're no longer in use.
    // Note: Calling any method on destroyed images is undefined behaviour.
    collection.destroyImage(imageA);
    collection.destroyImage(imageB);
    collection.destroyImage(imageC);

Each class provides a simple interface for uploading data to it, retrieving its data and placing barriers as part of a
:doc:`/modules/memory/transfer/buffer_transaction`. There are some guidelines you must follow that apply to all types of
images:

Firstly, when performing multiple operations on the same image in different transactions, you must commit the
transactions in the order in which you staged the operations. Otherwise, commands are recorded with the wrong image
layouts, queue ownership, etc.

.. code-block:: cpp
    :caption: Transaction commit order.

    sol::MemoryManager& manager = ...;
    sol::Image2D2&        image = ...;

    auto transaction0 = manager.beginTransaction();
    auto transaction1 = manager.beginTransaction();

    // Place barrier in transaction0.
    constexpr sol::Image2D::Barrier barrier0{
        .dstFamily = manager.getComputeQueue(),
        .srcStage = VK_PIPELINE_STAGE_2_NONE_KHR,
        .dstStage = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        .srcAccess = VK_ACCESS_2_NONE,
        .dstAccess = VK_ACCESS_2_SHADER_WRITE_BIT,
        .dstLayout = VK_IMAGE_LAYOUT_GENERAL
    };
    image.barrier(*transaction0, barrier0, sol::BarrierLocation::BeforeCopy);

    // Place barrier in transaction1.
    constexpr sol::Image2D::Barrier barrier1{
        .dstFamily = manager.getGraphicsQueue(),
        .srcStage = VK_PIPELINE_STAGE_2_COMPUTE_SHADER_BIT,
        .dstStage = VK_PIPELINE_STAGE_2_FRAGMENT_SHADER_BIT,
        .srcAccess = VK_ACCESS_2_SHADER_WRITE_BIT,
        .dstAccess = VK_ACCESS_2_SHADER_READ_BIT,
        .dstLayout = VK_IMAGE_LAYOUT_READ_ONLY_OPTIMAL
    };
    image.barrier(*transaction1, barrier1, sol::BarrierLocation::BeforeCopy);

    // We must commit 1 before 0, otherwise the barriers are submitted in the wrong order.
    transaction0->commit();
    transaction1->commit();
    transaction1->wait();
    
Secondly, you cannot stage multiple operations on the same image in the same transaction. These must be spread across
multiple transactions. Note that images do support copying multiple subregions in the same operation.
