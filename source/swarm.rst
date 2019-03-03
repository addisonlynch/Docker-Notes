.. _swarm:


Docker Swarms
=============

The components of distributed applications are called **services**. In docker,
services are simply "containers in production", but in practice these could be
the frontend, database server, etc. A service runs one image and codifies what
that image does (how many replicas of that image are running, etc.).

docker-compose
--------------

The ``docker-compose.yaml`` file:

.. code-block:: yaml

    version: "3"
    services:
      web:
        # replace username/repo:tag with your name and image details
        image: username/repo:tag
        deploy:
          replicas: 5
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
          restart_policy:
            condition: on-failure
        ports:
          - "80:80"
        networks:
          - webnet
    networks:
      webnet:

This file tells docker the following:
    - Pull image from online repo
    - Create a service called web
    - Set up 5 replicas of this image
        - Each gets 10% of CPU, 50M of RAM
        - Each shares port 80 on a load-balanced network called webnet
    - Defines the new network webnet


Creating a Service
------------------

To create the desired service, we must first:

.. code-block:: shell

    $ docker swarm init

We can now run the app:

.. code-block:: shell

    $ docker stack deploy -c docker-compose.yaml getstartedlab

Now we can run ``docker service ls`` to see the running service

.. code-block:: shell

    $ docker service ls
    ID                  NAME                MODE                REPLICAS            IMAGE                    PORTS
    4mj7miodkwgj        getstartedlab_web   replicated          5/5                 username/repo:tag        *:80->80/tcp

Note that the service "web" which we created has been prepended with the name
"getstartedlab" which was given. We can also see each container (called a
**task**) which is running from within the service:

.. code-block:: shell

    $ docker service ps getstartedlab_web

    ID                  NAME                  IMAGE                    NODE                DESIRED STATE       CURRENT STATE        ERROR               PORTS
    a8cio0wt6afy        getstartedlab_web.1   username/repo:tag        default             Running             Running 3 days ago
    ez226v9dcedk        getstartedlab_web.2   username/repo:tag        default             Running             Running 3 days ago
    s8tr7489s9cn        getstartedlab_web.3   username/repo:tag        default             Running             Running 3 days ago
    me7vkve7uqh7        getstartedlab_web.4   username/repo:tag        default             Running             Running 3 days ago
    yxeadks6jrh2        getstartedlab_web.5   username/repo:tag        default             Running             Running 3 days ago


Updating/Changing a Service
---------------------------

Services can be updated in-place by changing the ``docker-compose.yaml`` file
and simply running ``docker stack deploy`` with the same arguments as before.
Containers do not need to be stopped and/or started again to complete this
process.

Shutting Down a Service
-----------------------

1.
.. code-block:: shell

    docker stack rm getstartedlab

2.
.. code-block:: shell

    docker swarm leave --force


