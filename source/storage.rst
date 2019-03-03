.. _storage:

Storage
=======

** TODO you can also use files as bind mounts **


Docker provides **volumes**, a sort of virtual disk to store data and share it. There are two types of volumes, **Persistent** and **Ephemeral**. Volumes are not a part of images, but rather are local to hosts.

Further, docker provides **bind mounts**, which have limited functionality compared to volumes, and are commonly used with entire services (or swarms) rather than invididual containers. However, they become quite useful for individual containers in certain scenarios

.. _storage.named-volumes:

Named Volumes
-------------


Creating Named Volumes
~~~~~~~~~~~~~~~~~~~~~~

We can create arbitrary volumes using the following command:

.. code-block:: shell

    $ docker volume create <VOLUME-NAME>

This will create a new volume inside of /var/lib/docker/volumes, which we can see from the command ``docker volume list``:

.. code-block::shell

    DRIVER              VOLUME NAME
    local               volume1

The newly-created volume can be referenced by simply passing its name:

.. code-block::shell

    $ docker run -ti --rm -d --name tester -v volume1 ubuntu

The newly-created docker container will have a folder named ``volume1`` in its root directory, which is the mounted volume. We can also specify a directory within the container to mount the volume to:

.. code-block:: shell

    $ docker run -ti --rm -d --name tester -v volume1:/shared-data ubuntu

``volume1`` can now be accessed through the ``shared-data`` directory within the container.

Managing Named Volumes
~~~~~~~~~~~~~~~~~~~~~~

There are 5 major commands to manage named volumes

1. ``docker volume create`` - creates a volume
2. ``docker volume inspect`` - inspects a volume
3. ``docker volume ls`` - lists all volumes
4. ``docker volume prune`` - removes all unused local volumes
5. ``docker volume rm`` - removes a volume

Sharing Data
------------

When creating a volume, a new directory is created on the *host* machine within docker's storage directory, and docker manages the contents of this directory.

Between docker VM and container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sharing data between the host and the containers is easy with the use of Volumes.

**MacOs and Windows**

We need to connect to the Linux virtual machine to have a terminal running on the linux host

1. Connect to the machine via SSH

.. code-block:: shell

    $ docker-machine ssh

2. Create a folder ``temp``

.. code-block:: shell

    $ mkdir temp
    $ pwd temp

We're now going to take the result of the ``pwd`` command and use it as an argument when creating a new container

**Option 1: Specify host directory**

3. Create a new container

.. code-block:: shell

    $ docker run -ti --rm --name tester -v /home/docker/temp:/shared-folder ubuntu bash

Now, inside of the new container, we will have ``shared-folder`` which connects to the folder ``/home/docker/temp`` on the docker virtual machine. Creating a file in this folder will create a file in ``/home/docker/temp``.

**Option 2: Don't specify host directory**

3. Create a new container

.. code-block:: shell

    $ docker run -ti -d --rm --name tester2 -v /shared-folder-2 ubuntu

Now if we run the command ``docker inspect tester2``, we'll come across the ``mounts`` key:

.. code-block:: json

    "Mounts": [
    {
        "Type": "volume",
        "Name": "e270465b77cd981aa6655479bb5a199134fe55b657b14c1c0413a874498f0cfb",
        "Source": "/mnt/sda1/var/lib/docker/volumes/e270465b77cd981aa6655479bb5a199134fe55b657b14c1c0413a874498f0cfb/_data",
        "Destination": "/shared-folder2",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }

Here, docker has automatically created a subdirectory of the docker storage directory on our host machine. Docker will manage this folder for us and delete it after all containers which mount it are removed.

This method is recommended, as the host root directory used in the first step is obviously not inherently *portable*, so it's better to let docker manage the storage on its own.


Between Containers
~~~~~~~~~~~~~~~~~~

For this we'll use the ``volumes-from`` argument to ``docker run`` which will create a shared volume which will only exist in the scope that it is needed.

.. code-block:: shell

    $ docker run -ti -v /shared-data --name test1 ubuntu bash


So we can now start a second container using the ``volumes-from`` parameter to share this ``shared-data`` folder:

.. code-block:: shell

    $ docker run -ti --volumes-from test1 ubuntu bash

This new container will contain the ``shared-data`` folder, which it can add to or remove from. When the new container is killed, the files placed in ``shared-data`` will still exist in the ``test1`` container.

The volume (including the ``shared-data`` folder) will remain until the final container which is using it is killed.

.. _storage.bind-mounts:

Bind Mounts
-----------

A **bind mount** is the process of mounting a directory from the host machine to the inside of a docker container. This directory is **not** (unlike volumes) created on-demand on the host machine if it does not already exist.

.. note:: Bind mounts **can not** be accessed or modified from the docker CLI. Consider using `named volumes <https://docs.docker.com/storage/volumes/>`__ instead.



Read Only
~~~~~~~~~

We can also specify that a volume or bind mount is **read-only** by passing ``readonly`` or ``ro`` when they are coreated

New Volume
~~~~~~~~~~

Let's try:

.. code-block:: shell

    $ docker run -ti --rm -d --name test --mount destination=/shared-data,readonly ubuntu

The result?

.. code-block:: shell

    docker: Error response from daemon: invalid mount config for type "volume": must not set ReadOnly mode when using anonymous volumes.
    See 'docker run --help'.

This makes sense, as we would never create a new volume on the host machine only to set it to read-only.


Named Volume
~~~~~~~~~~~~

.. code-block:: shell

    $ docker run -ti --rm -d --name test --mount src=volume1,destination=/shared-data,readonly ubuntu


Bind Mount
~~~~~~~~~~

.. code-block:: shell

    $ docker run -ti --rm -d --name test --mount type=bind,src="$(pwd)"/temp,destination=/shared-data,readonly ubuntu
