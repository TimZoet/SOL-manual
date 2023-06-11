sol.material
============

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    material/common
    material/materials
    material/compute
    material/graphics
    material/ray_tracing

The :code:`sol.material` module contains classes for defining materials. Additionally, there are several classes that
define interfaces for managing sets of materials and material instances.

As shown in the below diagram, there are 4 distinct categories of classes in this module. There are the base classes
which define shared functionality. And there are the classes that correspond to the compute, graphics and ray tracing
pipelines.

.. figure:: /_static/images/material/material_class_diagram.svg
    :alt: Diagram of the material class hierarchy.
