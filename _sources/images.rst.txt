.. _images:

Images
======


- You cannot sum the size of the images to understand the total amount of space
  that docker is using
  - docker is even more space efficient than the listed number (usually)


.. _images.tags:

Tags (Versioning)
-----------------

We can have the same image with multiple ``TAGS`` by specifying a tag (which is
usually a version) when committing. Say we have a container with the ID
``af407c0ef7c8`` that we would like to create an image from. We can do this
with ``docker commit``:

.. code-block:: shell

    $ docker commit af407c0ef7c8 myimage

This has created an image ``myimage`` from the container ``af407c0ef7c8``. But
what if the app from the container has changed, but we dont want to rename the
next image entirely? Surely, theres a way to retain the ``myimage`` name but
only change the tag. We can do this by:

.. code-block:: shell

    $ docker commit af407c0ef7c8:v2.1 myimage

Now, when we run ``docker images`` we'll have:

.. code-block:: shell

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    myimage             v2.0                1442b904b529        3 seconds ago       223MB
    myimage             latest              e0381948613c        12 seconds ago      223MB

So we see that both images have the name ``myimage``, but with different tags.
We can access these images by:


.. _images.removing:

Removing Images
---------------

Docker images can accumulate quickly, and take up a lot of space. The docker
command ``docker rmi image:tag`` removes the image ``image`` with tag ``tag``.


.. _images.building:

Building Images (Dockerfiles)
-----------------------------

**Dockerfiles** are small programs which describe to docker how to build an
image.

.. code-block:: shell

    $ docker build -t <DESIRED-NAME> .

In this example, the current directory ``./`` is the location of the
dockerfile. Each line of a dockerfile takes in the state of the previous image
and creates a new one - it **does not** delete the existing image. Thus, each
step of running a dockerfile is cached. If the dockerfile is unchanged and a
container has already been created, the
