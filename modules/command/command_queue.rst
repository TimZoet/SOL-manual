Command Queue
=============

The :code:`sol::CommandQueue` class manages a list of commands, organized as a directed, acyclic graph. After the
commands have been added and their settings and dependencies defined, they can be repeatedly retrieved and executed, for
example as part of a render loop:

.. code-block:: cpp

    auto commandQueue = std::make_unique<sol::CommandQueue>();

    // Configure several commands.
    auto& command0 = commandQueue->createCommand<...>();
    command0.set...();
    auto& command1 = commandQueue->createCommand<...>();
    command1.set...();
    auto& command2 = commandQueue->createCommand<...>();
    command2.set...();
    auto& command3 = commandQueue->createCommand<...>();
    command3.set...();

    // Create dependencies between commands.
    //               -- command1 -- 
    //             /                \
    // command0 --                    -- command3
    //             \                /
    //               -- command2 --
    command1.addDependency(command0);
    command2.addDependency(command0);
    command3.addDependency(command1);
    command3.addDependency(command2);

    // Finalize queue and all added commands. Will throw if any commands have been configured incorrectly.
    commandQueue->finalize();

To fully run all commands in a queue, first call the :code:`start` method exactly once. Then, repeatedly call the 
:code:`getNextCommand` method. It will return a function object that can be invoked to execute a command. This method
can be called from multiple threads at once. If there are no more commands to execute, it will simply return an empty
object. Once all commands have been retrieved, call the :code:`end` method to reset the queue. After that, the queue can
be reused:

.. code-block:: cpp

    while (someCondition())
    {
        // Start command queue.
        commandQueue->start();

        // Retrieve first command.
        auto nextCmd = commandQueue->getNextCommand();

        while (nextCmd)
        {
            // Execute command.
            (*nextCmd)();

            // Get next command. May return an empty object, meaning there are no more commands.
            nextCmd = commandQueue->getNextCommand();
        }

        // All commands have been executed. Call end() to reset queue.
        commandQueue->end();
    }

Internally, the command queue uses a :code:`std::barrier` for each command to block its execution until all of its
dependencies have completed.
