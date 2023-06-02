sol.core
========

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    core/attachment
    core/buffer
    core/bottom_level_acceleration_structure
    core/command_buffer
    core/command_buffer_list
    core/command_pool
    core/compute_pipeline
    core/descriptor_pool
    core/descriptor_set_layout
    core/device
    core/device_memory
    core/fence
    core/framebuffer
    core/graphics_pipeline
    core/image
    core/image_view
    core/instance
    core/memory_allocator
    core/physical_device
    core/queue
    core/queue_family
    core/ray_tracing_pipeline
    core/render_pass
    core/render_pass_layout
    core/sampler
    core/semaphore
    core/shader_binding_table
    core/shader_module
    core/subpass
    core/surface
    core/swapchain
    core/top_level_acceleration_structure

The :code:`sol.core` module contains many classes to manage various Vulkan objects. You can look at it as a slightly
more convenient C++ layer built on top of the Vulkan C API. Most classes (with the exception of some utilities) wrap a
Vulkan handle, i.e. all the :code:`Vk<Something>` types. The functionality of the classes is kept to a minimum. More
advanced or high-level features are implemented in other modules.

All other modules are built on top of :code:`sol.core`. As such, there is no way to use this library without this
module.

Object Creation
---------------

Creating objects follows a pattern very similar to the Vulkan API. Each class has a corresponding :code:`Settings` 
struct. These structs have a number of member variables that specify the details of the object being created. A 
:code:`Settings` instance can be passed to the static :code:`create` and :code:`createShared` methods to create either a
:code:`std::unique_ptr` or :code:`std::shared_ptr`:

.. code-block:: cpp

    // Fill settings object.
    sol::VulkanImage::Settings settings;
    settings.format = VK_FORMAT_R8G8B8A8_UINT;
    ...

    // Create identical images using same settings object.
    auto image0 = sol::VulkanImage::create(settings);
    auto image1 = sol::VulkanImage::create(settings);

    // Create image with the same values for all properties but the format.
    settings.format = VK_FORMAT_R32_SFLOAT;
    auto image2 = sol::VulkanImage::create(settings);

    // Create image with settings from another object.
    auto image3 = sol::VulkanImage::create(image0->getSettings());

    // Create shared_ptr instead of the default unique_ptr.
    auto image4 = sol::VulkanImage::createShared(settings);

Whenever a :code:`Settings` object needs a reference to some other Vulkan object, instead of the :code:`Vk<Something>`
handle you can assign any kind of pointer or reference to it (note that this does not have any effect on the lifetime of
the involved objects):

.. code-block:: cpp

    sol::VulkanDevice::Settings devSettings;
    devSettings       = ...;
    auto device       = sol::VulkanDevice::create(devSettings);
    auto sharedDevice = sol::VulkanDevice::createShared(devSettings);

    // Assign to settings by unique_ptr, reference, raw pointer or shared pointer.
    sol::VulkanImage::Settings imgSettings;
    imgSettings.device = device;
    imgSettings.device = *device;
    imgSettings.device = device.get();
    imgSettings.device = sharedDevice;

Lifetimes
---------

Each object manages the lifetime of the Vulkan object it wraps, and no more than that. It is your responsibility to make
sure that e.g. any resources allocated on a particular Vulkan device are destroyed before that device goes out of scope.

.. code-block:: cpp

    // Bad!
    sol::VulkanImagePtr image;
    sol::VulkanDevicePtr device;

    {
        sol::VulkanDevice::Settings settings;
        settings = ...;

        device = sol::VulkanDevice::create(settings);
    }

    {
        sol::VulkanImage::Settings settings;
        settings.device = device;
        settings        = ...;

        image = sol::VulkanImage::create(settings);
    }

    // Device will go out of scope before image.
    ~device;
    ~image;

Caching Settings
----------------

By default, each object will keep around the settings with which it was created, and allows retrieving it through the
:code:`getSettings` method. You can use this to inspect an object, or perhaps for easily cloning them. However, storing
all these settings instances obviously results in a considerable amount of overhead. To that end, there is the option
to disable this, at which point all objects will only store the minimal set of required properties. Refer to the build
instructions for more information.
