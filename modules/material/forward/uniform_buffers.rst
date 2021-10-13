Uniform Buffers
===============

Consider a forward material with the following shaders:

.. code-block:: c

    // Vertex shader:

    layout(binding = 1) uniform UBO1
    {
        mat4 value;
    } ubo1[3];

    ...

    // Fragment shader:

    layout(binding = 0) uniform UBO0
    {
        vec4 value;
    } ubo0;

    layout(binding = 2) uniform UBO2
    {
        vec4 value0;
        vec4 value1;
    } ubo2;

    ...

This material's layout can be defined as follows:

.. code-block:: cpp

    MyMaterial::MyMaterial(...)
    {
        layout.addUniformBuffer(0, sizeof(float) * 4, 1, VK_SHADER_STAGE_FRAGMENT_BIT, 0);
        layout.addUniformBuffer(1, sizeof(float) * 16, 3, VK_SHADER_STAGE_VERTEX_BIT, 0);
        layout.addUniformBuffer(2, sizeof(float) * 4 * 2, 1, VK_SHADER_STAGE_FRAGMENT_BIT, 0);
        layout.finalize();
    }
