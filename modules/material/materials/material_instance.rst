Material Instance
=================

The :code:`sol::MaterialInstance` class is the base class of all pipeline type specific (i.e. compute, graphics and ray
tracing) material instance classes. Material instances hold the textures, uniform buffers, acceleration structures,
etc., that are to be bound for subsequent draw calls and dispatches.

In Vulkan, resources used by a pipeline are grouped into descriptor sets. :code:`SOL` closely follows this model. A
material instance holds the resources for a single descriptor set. A full material, and thus a full pipeline, is
represented by a distinct material instance for each descriptor set.

Much akin to the :code:`sol::Material` class, defining material instances can be done in a myriad of ways. However,
each material instance is required to do 2 things:

Firstly, each material instance must implement the :code:`getSetIndex` method. This method should return the (constant)
index of the particular descriptor set the instance represents.

Secondly, the required resource retrieval methods must be implemented. For example, if the descriptor set contains a
storage buffer, the :code:`getStorageBufferData` method must be overridden. There are default implementations for all
retrieval methods that throw an exception. If a descriptor set does not contain a certain type of resource, the matching
method should never be called.

.. note::

    TODO: Describe all the resource retrieval methods.
