luna.command
============

.. toctree::
    :maxdepth: 2
    :titlesonly:
    :hidden:

    command/command_queue
    command/i_command
    command/present
    command/render
    command/other

The :code:`luna.command` module contains classes for setting up a graph of commands for traversal, rendering, 
synchronization, etc. Commands implement the :code:`sol::ICommand` interface and can be added to a 
:code:`sol::CommandQueue` which supports multithreaded retrieval of the next available command, based on the completion
state of dependencies. There already are many classes that implement the  interface, and you can of course add your own.
