Queue
=====

The :code:`sol::VulkanQueueFamily` class manages the lifetime of a :code:`sol::VkQueue`. The 
:code:`sol::VulkanDevice` keeps a list of instances of this class, based on how many were requested during device
creation.

.. note::
    Queue interface needs to be improved a bit before it can be properly documented.
