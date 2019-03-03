.. _cloud:


Docker Cloud
============

.. _cloud.basics:

Basics
------

Docker Hub is a place to store images.

First create an account at `Docker Cloud <https://cloud.docker.com/>`__,
then loging using:

.. code-block:: shell

    $ docker login

This connects the docker hub account to the local machine.


Pushing Images
~~~~~~~~~~~~~~

It's easy to push an image to docker hub. This can be done using an existing image (say ``image1``) by creating first creating a new tag:

1. Tag the desired image

.. code-block:: shell

    $ docker tag image1 <USERNAME>/firsttry:part1

2. Push it to Docker Hub

.. code-block:: shell

    $ docker push <USERNAME>/firsttry:part1


Pulling Images
~~~~~~~~~~~~~~

We can now pull the image taht we just pushed and run it in one command:

.. code-block:: shell

    $ docker run -ti -p 8000:80 addisonlynch/firsttry:part1


