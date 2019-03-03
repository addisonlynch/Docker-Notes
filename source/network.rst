.. _networking:


Docker Networking
=================

Docker allows multiple private networks on your machine through containers

.. _networking.ports:

Exposing ports
--------------

- Explicitly specify ports of a container to expose to the computer or
  another container


.. code-block:: shell

    $ docker run --name simplerelay -ti -p 8001:8001 -p 8002:8005 ubuntu bash

This creates a container and says that we would like the container to listen on
ports 8001 and 8002, while making them available outside the container at ports
8001 and 8005, respectively

A nice way to test this container is to setup a relay between port 8001 and
8002, where all input to 8001 is piped to 8002. So, from the bash terminal of
the simplerelay container we can run the following command using netcat:

.. code-block:: shell

    $ nc -lp 8001 | nc -lp 8002

Now that we've set up a simple relay, we'll confirm that it's working.

So from the host machine (outside the container), we'll first figure out the IP
address of our running docker virtual machine by running the command
``docker-machine ip``

This will give us an IP address (192.168.99.100, for example) that we can then
connect to and pass a command

.. code-block:: shell

    $ nc 192.168.99.100 8001

From a separate terminal, we'll do the same for port 8002

.. code-block:: shell

    $ nc 192.168.99.100 8002

Now, if we enter text in the first terminal, it will appear in the second!

But what if our machine (like many) does not have netcat installed by default?
We'll simply start a throwaway container with a distribution
(such as Ubuntu 14.04) which does have netcat! This is the beauty of docker

.. code-block:: shell

    $ docker run -ti --rm ubuntu:14.04 bash

Boom!

Dynamic Exposure
~~~~~~~~~~~~~~~~

- Instead of specifying both the inside and the outside port for docker to
  expose, we can allow docker to handle this dynamically
- Docker will find an open port on the host machine and forward the desired
  port from the container to it

.. code-block:: shell

    $ docker run --name foo -d -ti -p 8043 -p 8044 ubuntu
    $ docker port foo

The ``docker port`` command will print the outside port that was exposed, then
we can connect to this port in the same way as before

.. _networking.communicating:

Communicating Between Containers
--------------------------------


Over the Network
~~~~~~~~~~~~~~~~

One easy way to communicate between containers ``foo`` and ``bar`` is to simply
have them connect to to the same port on the docker virtual network.

1. Create a container ``foo`` which will expose a port

.. code-block:: shell

    $ docker run -ti --rm --name foo -p 1234:1234 ubuntu:14.04 bash
    $ nc -lp 1234

2. Create a second container ``bar`` which needn't expose a port

.. code-block:: shell

    $ docker run -ti --rm --name bar ubuntu:14.04 bash
    $ nc 192.168.99.100 1234

You can now communicate between ``foo`` and ``bar``!


Linking
~~~~~~~

.. note:: This is generally used with an orchestration tool to handle operations. This should be used for services which are **on the same machine** (such as a server and its status monitor)

1. Spin up a container ``server``

.. code-block:: shell

    $ docker run -ti --rm --name server ubuntu:14.04 bash
    $ nc -lp 1234

2. Spin up a container ``client``

.. code-block:: shell

    $ docker run -ti --rm --link server --name client

By passing ``server`` to the ``--link`` flag, we have directly connected
``client`` to ``server``, and docker will automatically place ``server``'s
IP address in /etc/hosts. So from ``client`` we can easily connect with a
command such as:

.. code-block:: shell

    $ nc server 1234

And we have connected!

.. _networking.virtual:

Virtual Networks
----------------

Docker uses private networks, which have built in nameservers. These networks
must be manually created using ``docker network create``. Docker provides 2
built-in network drivers, mainly ``bridge`` and ``overlay`` to enable
customzied communicatoin between containers. There are three networks by
default in every docker installation:

.. code-block:: shell

    NETWORK ID          NAME                DRIVER              SCOPE
    89ff83a4a99f        bridge              bridge              local
    f60cfebb22a2        host                host                local
    61d34cbc04f0        none                null                local

Docker places all containers on the ``bridge`` network by default.

Connecting 2 Containers
~~~~~~~~~~~~~~~~~~~~~~~

The problem with the examples of linking above is that the link may break if
the ``server`` container is killed. This would obviously not work out well for
the ``client`` container. To connect two containers over a new private network,
which we'll call ``example``, we'll follow these steps:

1. Create the network

.. code-block:: shell

    $ docker network create example

2. Create a new ``server`` container, and connect it to this network with the
   ``--net`` flag

.. code-block:: shell

    $ docker run -ti --rm --name server --net=example ubuntu:14.04 bash
    $ nc -lp 1234

3. Create a ``client`` container, connecting to the same network

.. code-block:: shell

    $ docker run -ti --rm --name client --link server --net=example ubuntu:14.04 bash
    $ np server 1234

The communication between the two containers will now remain possible
(unlike the previous method which did not use the ``example`` network) even if
the container ``server`` shuts down and restarts.



