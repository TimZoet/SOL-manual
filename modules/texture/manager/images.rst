Images
======

The :code:`sol::TextureManager` has various methods to create images, which in turn have methods to modify the image
data and properties. These are described in their respective sections. However, it is the manager that is in charge of
actually filling the image buffers with data and managing queue family ownership. For that, it has an instance of the
:code:`IImageTransfer` class.

There is currently one implementation, the :code:`DefaultImageTransfer`. When constructing a
:code:`sol::TextureManager`, it automatically creates an instance of this class, although this can be overridden by
passing an explicit transfer object to the constructor:

.. code-block:: cpp

    // Some custom class that implements the IImageTransfer interface.
    class MyImageTransfer : public IImageTransfer
    {
        ...
    };

    // Constructing a texture manager with a custom transfer object.
    auto myTransfer = std::make_unique<MyImageTransfer>(...);
    auto textureManager = std::make_unique<sol::TextureManager>(*memoryManager, std::move(myTransfer));

At present, the texture manager has only a single method relevant to performing copies, :code:`transferStagedCopies`.
This invokes the transfer object which will execute copy commands and place image memory barriers. Combined, these
commands take care of any required image layout transitions and queue family ownership transfers, as well as the actual
copying of data from the staging buffers to the image buffers. 

.. code-block:: cpp

    // Create a bunch of images. This will create a staging buffer for each.
    math::ubyte4 white{255, 255, 255, 255};
    auto& image0 = textureManager->createImage2DFill(VK_FORMAT_R8G8B8A8_SRGB, {256, 256}, &white);
    auto& image1 = ...;
    
    // Perform copies.
    textureManager->transferStagedCopies();

Calling :code:`transferStagedCopies` should only be done when the resources are not being used. In the above example,
no synchronization is required because the images are clearly not in use. However, in the likely scenario that you have
some kind of render loop, this should be synchronized outside of the manager.