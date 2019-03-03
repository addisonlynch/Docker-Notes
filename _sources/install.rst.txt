.. _installing:

Installing Docker
=================

- Docker Community Edition (CE) is common for single-users

Use the below methods then test to see if the install worked using:

.. code-block:: shell

    $ docker run hello-world


.. _installing.mac:

Docker Install (Mac)
--------------------

- Docker will need its own virtual machine to run on a Mac or windows
- This is unlike linux which can just use the host machine


1. Go to docker's site and download the Docker Toolbox for MacOS

2. Install package

3. Spotlight and run "Docker Quickstart Terminal"
    - This will setup the docker server

4. Run the command:

.. code-block:: shell

    $ eval "$(docker-machine env default)"

5. We can now run docker-compose to show a number of command which can be used
to manage the docker server

.. _installing.ubuntu:

Docker Install (Ubuntu)
-----------------------

- Follow the instructions https://docs.docker.com/install/linux/docker-ce/ubuntu/
    - These frequently change

1. Set up the repository

.. code-block:: shell

    $ sudo apt-get update

    $ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    $ sudo apt-key fingerprint 0EBFCD88

    $ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

    $ sudo apt-get update

    $ sudo apt-get install docker-ce
