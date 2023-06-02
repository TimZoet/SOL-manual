Bottom Level Acceleration Structure
===================================

The :code:`sol::VulkanBottomLevelAccelerationStructure` class manages the lifetime of a
:code:`VkAccelerationStructureKHR` and the accompanying :code:`sol::VulkanBuffer` that holds all acceleration structure
memory.

.. note::

    TODO: Currently, building the BLAS is done inside of this class. This functionality is going to be moved to a
    separate module dedicated to building acceleration structures. This class will then only manage the handle and
    buffer.
