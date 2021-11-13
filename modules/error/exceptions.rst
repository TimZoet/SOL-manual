Exception Classes
=================

All exceptions directly thrown by this library are of type (or derived from) :code:`sol::SolError`, which itself derives
from :code:`std::exception`. Most exceptions thrown are of type :code:`sol::SolError`.

There is also a special :code:`sol::VulkanError` exception (as well as some even more specific ones) which are thrown 
when a call to a Vulkan API function fails.

.. figure:: /_static/images/exception_class_diagram.svg
    :alt: Diagram of the exception class hierarchy.
