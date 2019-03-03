√.. _basics:

Docker Basics
=============


This section focuses on the ``docker run`` command, which turns **images** into running **containers**.

.. _basics.image:

Image
-----

- Just enough of the operating system to do what you need to do
- You can have lots and lots of these on the computer

.. code-block:: shell

    $ docker images

- This will show all docker images, including:
    - Repository
    - Tag
    - Image ID
    - Date Created
    - Size

What do we use Images for?
~~~~~~~~~~~~~~~~~~~~~~~~~~

- We "run" images with ``docker run`` to turn an image into a **Container**

Example:

.. code-block:: shell

    $ docker run -ti ubuntu:latest bash

This will run an image of the latest version of Ubuntu
- The ``-ti`` flag tells docker that we wish to use a shell with this image
- ``bash`` automatically enters the shell for the running Docker image
- When we make a container from an image, **that image does not change**.

.. _basics.containers:

Containers
----------

Once we have an image running inside of a container (as above), we can now use ``docker ps`` to see the running container on our system:

.. code-block:: shell

    $ docker ps

This will output each running container by its ID, which is different from the image ID.


Files
~~~~~

For practice, create the ``ubuntu:latest`` image and create a file from a shell of that image:

.. code-block:: shell

    $ touch hello

If we create a new container from this same image (``ubuntu:latest``) and create the same file in the same location, it will not be visible in the other container.

Stopped Containers
~~~~~~~~~~~~~~~~~~

- Alive containers have a running process in them
- When this process ends, the container **stops**

By default, ``docker-ps`` only displays the containers which are *running*. To view all containers, use:

.. code-block:: shell

    $ docker ps -a

We can view the most recently-exited container as well:

.. code-block:: shell

    $ docker ps -l

So what can we do with stopped containers? Say we have a file inside a stopped container that we would like to access after it has stopped


Docker Flow: Containers to Images
---------------------------------

This section focuses on the ``docker commit`` command, which enables the creation of **images** from **containers**. Say we've created a file in a running container that we would like to retrieve after that container is stopped. What can we do?

- This **will not** overwrite an existing image

1. Create a new Ubuntu container and create an arbitrary file

.. code-block:: shell

    $ docker run -ti ubuntu bash
    $ touch HELLO

2. Exit the container using ctrl-D or ``exit``

3. Obtain the container ID through ``docker ps -l``

4. Create an image from the stopped container using ``docker commit``:

.. code-block:: shell

    $ docker commit <CONTAINER-ID>

The outcome of this command will be a long hash signature (``sha256:``), which
can then be used to name the created image with ``docker tag``

.. code-block:: shell

    $ docker tag <HASH-TAG> <DESIRED-NAME>

OR

We can just specify the desired name when we commit in the first place

.. code-block:: shell

    $ docker commit <CONTAINER-ID> <DESIRED-NAME>

The latter method is the easiest, but it is necessary to show the tag command
for an understanding of what's going on behind the scenes. We now have an image
named DESIRED-NAME.

5. Run a container from the new image to see your file

.. code-block:: shell

    $ docker run -ti <DESIRED-NAME> bash
    $ ls


.. _basics.commands:

Docker Commands
---------------

Attaching/Detaching Containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


- ``docker-run`` - starts a docker container, which has a main process
    - Containers stop when that main process stops
        - Even if you run other processes, it waits for the main process to stop
    - ``docker run --rm``
        - This says that after the command is run, get rid of the container
        - This is good for one time use
        - Example: start a container that will sit there for 5 seconds and then exit

        .. code-block:: shell

            $ docker run --rm -ti ubuntu sleep 5

        - We could also run a more fancy container:

        .. code-block:: shell

            $ docker run --rm -ti ubuntu "sleep 3; echo "all done""

        - This will start a container that will run for 3 seconds and then print
          "all done"
    - ``docker run -d``
        - This will start a **detached** container. We can retrieve this container ID
          from ``docker ps``
        - We can then attach the detached container by obtaining its container
          ID and running ``docker attach <CONTAINER-ID>``
        - To exit a container by detaching it instead of stopping it, use
          ctrl-P + ctrl-Q

Killing/Removing a Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can kill an existing container using the command:

.. code-block:: shell

    $ docker kill <CONTAINER-ID>

We can also kill and remove the container completely in one step using:

.. code-block:: shell

    $ docker rm <CONTAINER-ID>



Starting a new process
~~~~~~~~~~~~~~~~~~~~~~

The command ``docker exec`` can start a new process in an existing (running)
container.


Container Output (Logs)
-----------------------

We can capture the output of a container using the ``docker log`` command. Let's
create a container whose main process is going to fail:

.. code-block:: shell

    $ docker run --name bad -ti ubuntu bash -c "lose /etc/password"

We've created a Ubuntu container ``bad`` which will attempt to execute the command
``lose /etc/password``. Given that this is an invalid bash command (we have
probably misspelled 'less'), it will immediately exit with code 1 and the container
will be killed. So we try:

.. code-block:: shell

    $ docker logs bad

This will show us ``bash: lose: command not found``, which is the reason that the
container was killed!

.. _basics.resources:

Container Resources
-------------------


Memory Constraints
~~~~~~~~~~~~~~~~~~

.. code-block:: shell

    $ docker run --memory maximum-allowed-memory

CPU Constraints
~~~~~~~~~~~~~~~

We can limit CPU usage relative to another container
- "If one container is not using any resources, allow the other to have them.
If they are both running, split the resource 50/50 if both containers are using
an equal amount of the resource"

.. code-block:: shell

    $ docker run --cpu-shares

We can also set a hard limit on CPU usage

.. code-block:: shell

    $ docker run --cpu-quota

Many orchestration systems force you to set resource limits on containers

.. _basics.tips:

General Tips
------------

- Do not allow docker containers fetch dependencies when they start
    - If the dependencies change or are removed, the containers will all die
- Do not leave stopped unnamed containers sitting around
    - Security issue: you or someone else may just delete all the containers to
      save space or attack
