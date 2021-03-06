# Distributed Docker Hadoop

This repository demonstrates how to spin up a distributed Hadoop system.

* [Prerequisites](#prerequisites)
* [Setup](#setup)
  * [Build Docker image:](#build-docker-image)
  * [Start cluster](#start-cluster)
  * [Scaling out](#scaling-out)
* [Moving to multiple hosts](#moving-to-multiple-hosts)

## Prerequisites

Ensure you have Python Anaconda (the Python 3.6 flavor) installed: https://www.anaconda.com/download/.
Further ensure you have a recent version of Docker installed.
The Docker version I developed this example on is:

    $ docker --version
    Docker version 17.05.0-ce, build 89658be

## Setup

We will use Docker Compose to spin up the various Docker containers constituting
our Hadoop system.
To this end let us create a clean Anaconda Python virtual environment and install
a current version of Docker Compose in it:

    $ conda create --name distributed_docker_hadoop python=3.6 --yes
    $ source activate distributed_docker_hadoop
    $ pip install -r requirements.txt

Make certain `docker-compose` points to this newly installed version in the virtual
environment:

    $ which docker-compose

In case this does not point to the `docker-compose` binary in your virtual environment,
reload the virtual environment and check again:

    $ source deactivate
    $ source activate distributed_docker_hadoop

## Build Docker image

To build the Docker image our Hadoop components will run on:

    $ docker-compose build

## Start cluster

To start up the cluster:

    $ docker-compose up --force-recreate

Once all Docker services are up you can visit a couple of GUIs in your browser
to study the overall status of your cluster:

* [The name node](http://localhost:50070)
* [The resource manager](http://localhost:8088)
* [The MapReduce job history server](http://localhost:8088)

## Scaling out

Hadoop is well-known for allowing to scale out, i.e. easily run across numerous hosts.
Since we are using Docker Compose to spin up our virtual hosts in this toy example, we can
play around with scaling out by using the ability of Docker to scale up Docker services.

### Data nodes

Bring up the Hadoop cluster as described above.
Browse the current list of data nodes by visiting the web interface of the name node:

[http://localhost:50070/dfshealth.html#tab-datanode](http://localhost:50070/dfshealth.html#tab-datanode)

You should see a single data like so:

![image](https://user-images.githubusercontent.com/3273502/29886791-98f586f4-8dbb-11e7-9bbb-ca6d8314de2f.png)

In a separate terminal window activate the Python virtual environment and scale
up the data node service as follows:

    $ source activate distributed_docker_hadoop
    $ docker-compose up --scale data-node=2

Back in the name node web interface you should now notice two data nodes:

![image](https://user-images.githubusercontent.com/3273502/29886878-e00ef7a0-8dbb-11e7-8e91-54117244b115.png)

## Notes

### Hostnames

Hostnames are not allowed to contain underscores `_`, therefore make certain
to spell out longer hostnames with dashes `-` instead.
In this example we ensure this by using dashes in the names of our Docker services.

### Moving to multiple hosts

It should be relatively simple to scale out our test cluster to multiple hosts.
Here is a sketch of steps that are likely to get you closer to running this on multiple hosts:

* Create a [Docker swarm](https://docs.docker.com/engine/swarm/)
* Instead of the current Docker bridge network use an
  [overlay network](https://docs.docker.com/engine/userguide/networking/get-started-overlay/)
* Add an [OpenVPN server container](https://github.com/kylemanna/docker-openvpn) to your
  Docker overlay network to grant you continued web interface access from your computer
* You would likely want to use an orchestration framework such as
  [Ansible](https://www.ansible.com/) to tie the different steps and components
  of a more elaborate multi-host deployment together
