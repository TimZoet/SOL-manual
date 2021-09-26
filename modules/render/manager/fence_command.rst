Fence Command
=============

The :code:`sol::FenceCommand` can be used to wait for and/or reset one or more fences. When a single command is given
multiple fences to wait for or reset, these must all be part of the same device:

.. code-block:: cpp

    // Create a fence command to wait for a single fence.
    auto fenceCmd0 = std::make_unique<sol::FenceCommand>(sol::FenceCommand::Action::Wait, "myFence");

    // Create a fence command to wait for and reset multiple fences.
    auto fenceCmd1 = std::make_unique<sol::FenceCommand>(sol::FenceCommand::Action::Wait |
                                                         sol::FenceCommand::Action::Reset, 
                                                         {"fence0", "fence1"});

There are many use cases in which a fence command should wait for one of a number of fences. For example, if you have a
render loop where each iteration a different image is used to render to and there is a separate fence for each image. 
For that, it is possible to pass a function to the fence command to generate the fence(s). This function is invoked 
whenever the command is executed:

.. code-block:: cpp

    // Create a fence command to wait for a dynamic fence.
    uint32_t index = 0;
    auto func = [&index]() -> std::vector<std::string> { return { std::format("fence{0}", index) }; };
    auto fenceCmd2 = std::make_unique<sol::FenceCommand>(sol::FenceCommand::Action::Wait, func);
