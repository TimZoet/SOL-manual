Image2D
=======

The :code:`sol::Image2D` class manages a 2-dimensional, single layer image. The object itself is owned by the 
:code:`sol::TextureManager`. Internally it holds an instance of :code:`sol::VulkanImage`, which manages the actual 
Vulkan image resource handle.

New images can be created through the :code:`sol::TextureManager`. There are various methods to create 
image objects with a default fill color, from a data array, not yet initialized, etc. At minimum, the format and 
dimensions of an image must be specified on construction:

.. code-block:: cpp

    // Create an image with uninitialized data.
    auto& image0 = texManager.createImage2D(VK_FORMAT_R8G8B8A8_SRGB, {512, 512});

    // Create an image filled with all red values.
    std::array<uint8_t, 4> rgba = {255, 0, 0, 255};
    auto& image1 = texManager.createImage2DFill(VK_FORMAT_R8G8B8A8_SRGB, {256, 256}, rgba.data());

After construction, the image buffer can be updated using similar methods of the image object:

.. code-block:: cpp

    // Fill vector with enough data to update entire image.
    std::vector<uint8_t> colorData;
    ...

    // Initialize data.
    image0.setData(colorData.data());

Resizing
--------

.. note::
    Not yet implemented.

Mipmaps
-------

.. note::
    Not yet implemented.
