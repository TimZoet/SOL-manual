Buffer
======

The :code:`sol::VulkanBuffer` class manages the lifetime of a :code:`VkBuffer`. It can be created in two ways. 

The first way is by creating a buffer, followed by binding it to a (potentially newly created)
:code:`sol::VulkanDeviceMemory`. This of course requires you to specify all properties for both the buffer and memory.
For example, creating a vertex buffer and allocating some memory for it could be done as below:

.. code-block:: cpp
    
    // First create a buffer.
    sol::VulkanBuffer::Settings settings;
    settings.device      = device
    settings.size        = vertexCount * sizeof(Vertex);
    settings.bufferUsage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    settings.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
    auto buffer          = sol::VulkanBuffer::create(settings);

    // Then allocate memory.
    sol::VulkanDeviceMemory::Settings memSettings;
    memSettings.device              = device;
    memSettings.size                = buffer->getMemoryRequirements().size;
    memSettings.memoryTypeBits      = buffer->getMemoryRequirements().memoryTypeBits;
    memSettings.memoryPropertyFlags = VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT;
    auto memory                     = sol::VulkanDeviceMemory::create(memSettings);

    // Finally, bind memory to buffer.
    buffer->bindMemory(memory);

Alternatively, buffers can be created with the help of the :code:`VulkanMemoryAllocator` (:code:`VMA`) library. When 
using the :code:`VMA`, there is no need to explicitly allocate and bind memory, since this is handled by the library. 
All you need to do is specify an :code:`allocator`, as well as some additional properties that are forwarded to the 
allocator:

.. code-block:: cpp
    
    sol::VulkanBuffer::Settings settings;
    settings.device          = device
    settings.size            = vertexCount * sizeof(Vertex);
    settings.bufferUsage     = VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    settings.sharingMode     = VK_SHARING_MODE_EXCLUSIVE;
    settings.allocator       = allocator;
    settings.vma.memoryUsage = VMA_MEMORY_USAGE_GPU_ONLY;
    settings.vma.flags       = 0;
    auto buffer              = VulkanBuffer::create(settings);

Mapping
-------

The buffer can be mapped and unmapped using the :code:`map` and :code:`unmap` methods. These correspond directly to the
:code:`vkMapMemory` and :code:`vkUnmapMemory` functions, except when using an allocator. In that case, they use the
:code:`vmaMapMemory` and :code:`vmaUnmapMemory` functions which internally do some reference counting. Either way, it is
your responsibility to keep track of how many times the :code:`map` and :code:`unmap` methods are called. The 
:code:`isMapped` property can help with this.

Copying Data
------------

There are several :code:`setData...` methods with which data can be copied to a buffer. These methods do not take care 
of mapping, unmapping and flushing. This must be handled elsewhere.

As an example, the following code snippet roughly shows how to fill an 8-bit color image buffer with all green:

.. code-block:: cpp
    
    const std::array<uint8_t, 4> green = {0, 255, 0, 255};
    const uint32_t width               = 1024;
    const uint32_t height              = 1024;

    // Assuming a staging buffer for an R8G8B8A8 image.
    auto imageStagingBuffer = sol::VulkanBuffer::create(...);
    
    // Map, set data, unmap.
    imageStagingBuffer->map();
    imageStagingBuffer->setDataRepeated(green.data(), sizeof(uint8_t) * 4, width * height);
    imageStagingBuffer->unmap();

    // Probably transfer the staging buffer to the actual image buffer.
    ...
