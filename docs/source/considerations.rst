**************
Considerations
**************


Clients versus servers
======================

When using :ref:`basic_communication_patterns` we have a lot of flexibility:

- We are allowed to connect multiple clients to a server.
- The server can play any role (i.e.: does not need to be always ``REP``, but
  can be ``REQ`` as well). Servers are only defined by the action of binding,
  not by the role they play in the communication pattern.

For example, if we bind using ``PUSH`` and we connect multiple clients to
this server, then messages pushed will be distributed among the clients in a
`Round-robin fashion <https://en.wikipedia.org/wiki/Round-robin_scheduling>`_,
which means the first message will be received by the first client, the
second message will be received by the second client, and so on.

If we bind using ``PULL`` and we connect multiple clients to this server,
then messages pushed will all be received by the single server, as expected.

For more information simply refer to the
`ØMQ guide <http://zguide.zeromq.org/page:all>`_.


Adding new methods
==================

Note that proxies can not only be used to execute methods remotely in the
agent, but they can also be used to add new methods or change already
existing methods in the remote agent.

In the following example you can see how we can create a couple of functions
that are then added to the remote agent as new methods.

In order to add new methods (or change current methods) we only need to call
``set_method()`` from the proxy.

.. literalinclude:: ../../examples/add_method.py

Note that ``set_method()`` accepts any number of parameters:

- In case they are not named parameters, the function names will be used as
  the method names in the remote agent.
- In case they are named parameters, then the method in the remote agent will
  be named after the parameter name.


Lambdas
=======

osBrain uses dill for serialization when communicating with remote agents
through a proxy. This means that almost anything can be serialized to an agent
using a proxy.

In order to further simplify some tasks, lambda functions can be used to
configure remote agents:

.. literalinclude:: ../../examples/req_rep_lambda.py

See the similarities between this example and the one showed in
:ref:`request_reply`.
In fact, the only difference is the binding from Alice, in which we are using
a lambda function for the handler.


Shutting down
=============

Although not covered in the examples until now (because many times you just
want the multi-agent system to run forever until, perhaps, an event occurs),
it is possible to actively kill the system using proxies:

.. literalinclude:: ../../examples/shutdown.py

.. note:: Shutting down the nameserver will result in all agents registered in
   the name server being shut down as well.


OOP
===

Although the approach of using proxies for the whole configuration process is
valid, sometimes the developer may prefer to use OOP to define the behavior of
an agent.

This, of course, can be done with osBrain:

.. literalinclude:: ../../examples/push_pull_inherit.py

Most of the code is similar to the one presented in the :ref:`push_pull` example,
however you may notice some differences:

#. When runing *Alice*, a new parameter ``base`` is passed to the
   :func:`osbrain.run_agent` function. This means that, instead of
   running the default agent class, the user-defined agent class will be used
   instead. In this case, this class is named ``Greeter``.
#. The ``Greeter`` class implements two methods:

   #. ``on_init()``: which is executed on initialization and will, in this
      case, simply bind a ``'PUSH'`` communication channel.
   #. ``hello()``: which simply logs a *Hello* message when it is executed.

#. When connecting *Bob* to *Alice*, we need the address where *Alice* binded
   to. As the binding was executed on initialization, we need to use the
   ``addr()`` method, which will return the address associated to the alias
   passed as parameter (in the example above it is ``main``).


Setting initial attributes
==========================

Many times, after spawning an agent, we want to set some attributes, which
may be used to configure the agent before it starts working with the rest of
the multi-agent system:

.. code-block:: python

   a0 = run_agent('foo')
   a0.set_attr(x=1, y=2)

It is such a common task that a parameter ``attributes`` can be used when
running the agent for exactly that:

.. code-block:: python

   a0 = run_agent('foo', attributes=dict(x=1, y=2))

As you can see, this parameter accepts a dictionary in which the keys are the
name of the attributes to be set in the agent and the values are the actual
values that this attributes will take.
