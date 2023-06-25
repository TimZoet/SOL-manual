Graphics Material Instance
==========================

The :code:`sol::GraphicsMaterialInstance` class inherits from :code:`sol::MaterialInstance`. It adds additional methods
for defining dynamic state.

Dynamic State
-------------

If a certain type of pipeline state was defined as dynamic on the material layout, you can set the matching value on
material instances:

.. code-block:: cpp

    // Enable dynamic culling on the layout.
    sol::GraphicsMaterialLayout& layout = ...;
    layout.enableDynamicState<VK_DYNAMIC_STATE_CULL_MODE>(true);
    ...

    // Add dynamic state value to material instance.
    sol::GraphicsMaterialInstance& instance = ...;
    instance.setCullMode(sol::CullMode::Front);

When a draw call is being performed, all material instances that are bound must together define the full set of dynamic
state values. Not doing so will result in unexpected behaviour. There are no restrictions on which material instances
hold what state values. When multiple material instances are used, all dynamic state could be held by just one of them,
or it could be spread across them.
