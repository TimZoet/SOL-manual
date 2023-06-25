Materials
=========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    materials/material_layout_description
    materials/material_layout
    materials/material
    materials/material_instance

As shown in the below diagram, there are 4 distinct categories of classes in this module related to describing
materials. There are the base classes which define shared functionality. And there are the classes that correspond to
the compute, graphics and ray tracing pipelines.

.. figure:: /_static/images/material/material_class_diagram.svg
    :alt: Diagram of the material class hierarchy.

This section covers the shared functionality. The pipeline specific material classes each have their own section:

* :doc:`/modules/material/compute`
* :doc:`/modules/material/graphics`
* :doc:`/modules/material/ray_tracing`
