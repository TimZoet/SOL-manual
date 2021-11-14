Command Buffer
==============

The :code:`sol::VulkanCommandBuffer` class manages the lifetime of a single :code:`VkCommandBuffer`. It has a few 
convenience methods to begin, end and reset the command buffer.

Creating a command buffer requires only a command pool and optionally a command buffer level:

.. code-block:: cpp

    // Assuming a previously created command pool.
    sol::VulkanCommandPool& pool = ...;

    // Command buffers are primary by default.
    sol::VulkanCommandBuffer::Settings settings;
    settings.commandPool = pool;
    auto primaryBuffer   = sol::VulkanBuffer::create(settings);

    // To create a secondary command buffer specify its level explicitly.
    sol::VulkanCommandBuffer::Settings settings2;
    settings2.commandPool = pool;
    settings2.level       = VK_COMMAND_BUFFER_LEVEL_SECONDARY;
    auto secondaryBuffer  = sol::VulkanBuffer::create(settings2);

Starting and ending the buffer can be done directly. The methods are pretty self-explanatory. Obviously, to e.g. begin a
one time command and reset it, the command pool from which it was allocated must be compatible.

.. code-block:: cpp

    primaryBuffer->beginOneTimeCommand();
    ... // Record stuff.
    primaryBuffer->endCommand();
    primaryBuffer->resetCommand();

    secondaryBuffer->beginCommand();
    ... // Record stuff.
    secondaryBuffer->endCommand();
