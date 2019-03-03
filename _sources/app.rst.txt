.. _app:

.. note:: This heavily relies on the tutorial from Docker which can be found `here <https://docs.docker.com/v17.09/get-started/part2/>`__.

*********************
Building Applications
*********************

Dockerfiles
===========

A **Dockerfile** defines the environment of a container.

Here's an example:

.. code-block:: dockerfile

    # Use an official Python runtime as a parent image
    FROM python:3.7-slim

    # Set the working directory to /app
    WORKDIR /app

    # Copy the current directory contents into the container at /app
    ADD . /app

    # Install any needed packages specified in requirements.txt
    RUN pip install --trusted-host pypi.python.org -r requirements.txt

    # Make port 80 available to the world outside this container
    EXPOSE 80

    # Define environment variable
    ENV NAME World

    # Run app.py when the container launches
    CMD ["python", "app.py"]

