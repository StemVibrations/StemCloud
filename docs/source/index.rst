.. StemCloud documentation master file, created by
   sphinx-quickstart on Thu Jun 13 14:55:45 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to StemCloud's documentation!
=====================================

StemCloud is an open-source Python package that enables users to perform cloud calculations using  the `STEM package <https://stemvibrations.readthedocs.io/>`_ .
The cloud infrastructure is not provided by StemCloud, but the package provides a simple interface to run STEM calculations on the user's cloud infrastructure.

Cloud computing
===============

Cloud computing is the on-demand availability of computer system resources, especially data storage and computing power, without direct active management by the 
user. The term is generally used to describe data centers available to many users over the Internet. Large clouds, predominant today, often have functions 
distributed over multiple locations from central servers. 

In the context of STEM calculations, cloud computing can be used to run calculations on a remote server, which can be accessed from anywhere in the world.
This is particularly useful for large calculations that require a lot of computational power or take a lot of time to run, as the user can run the calculations 
on a powerful server without having to invest in expensive hardware. 

StemCloud
=========

StemCloud provides the user with the necessary tools to run STEM calculations on a remote server. This is done by creating a virtual machine on the cloud provider
of the user's choice, installing the necessary software, and running the calculations on the virtual machine.

Docker 
======

Docker is a tool that is used to automate the deployment of applications inside software containers. 
Containers are a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries, and settings.
Therefore, Docker is used to create a container that contains all the necessary software to run STEM calculations. 
This container can then be run on any machine that has Docker installed.

The default Dockerfile that is used to create the Docker container is shown below. This Docker container is designed to set up an environment 
for running a Python application related to the StemCloud project. The container uses a slim version of Python 3.10 as its base image to minimize size 
while still providing the necessary functionality. The Dockerfile includes steps to copy the project files and dependencies into the container, install 
necessary system packages, and ensure that all required Python libraries are installed. 



.. include:: _static/Dockerfile
   :literaL:


Installation and Usage 
---------------------- 

To actually run the docker container, you need to have Docker installed on your machine. You can download Docker from the `official website <https://docs.docker.com/engine/install/>`_.
After you have installed Docker, you can run the following command to build the Docker container:

.. code-block:: bash

    docker build -t stemcloud .

This command will build the Docker container using the Dockerfile provided in the project directory. 
The `-t` flag is used to tag the container with the name `stemcloud`.

After the container has been built, you can run the following command to start the container:

.. code-block:: bash

    docker run -it stemcloud

Depending on the infrastructure you are using, these commands may need to be modified slightly. For example, if you are using a cloud provider like AWS,
you may need to use the `docker-machine` command to create a virtual machine on the cloud provider and then run the Docker container on that virtual machine.


Single and pallarel calculations using Argo
============================================

This project leverages `Argo <https://argoproj.github.io/>`_ to orchestrate and manage complex workflows within a Kubernetes environment. 
Argo allows us to define, schedule, and monitor workflows in a Kubernetes-native way, ensuring efficient 
and scalable execution of our data processing and CI/CD pipelines. Kubernetes, often abbreviated as K8s, 
is an open-source platform designed for automating the deployment, scaling, and management of containerized applications. 

In essesnce, Argo allows us to define a workflow that consists of multiple steps, each of which can be run in parallel or sequentially.
This is particularly useful for running STEM calculations, as we can define a workflow that consists of multiple calculations that can be run in parallel
and then combine the results at the end. Especially for random field calculations, this can significantly reduce the time it takes to run the calculations.

However, Argo requires a Kubernetes cluster to run the workflows. This can be set up on a cloud provider like AWS, Google Cloud, or Azure, or on a local machine using Minikube.
For example using AWS the following steps can be followed to set up a Kubernetes cluster:

1. Install the `AWS CLI <https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html>`_ on your machine.
2. Log in to your AWS account using the `aws configure` command.
3. Install `kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`_ on your machine.
4. Install `eksctl <https://eksctl.io/introduction/installation/>`_ on your machine.
5. Update kubeconfig using the following command: `aws eks --region <region> update-kubeconfig --name <cluster_name>`.
6. Install Argo CLI folling the instraction on the `official website <https://github.com/argoproj/argo-workflows/releases/>`_.
7. Submit the workflow using the following command: `argo submit <workflow_file>`.
8. Monitor the workflow using the following command: `argo list`.
9. Get the logs of the workflow using the following command: `argo logs <workflow_name>`.
10. Delete the workflow using the following command: `argo delete <workflow_name>`.

The following is an example of a workflow that can be used to run a single STEM calculation using Argo.
The Argo workflow is defined to run a specific Python script located within a Docker container. 
The workflow captures the script's output and stores it in an S3 bucket. Below is a detailed breakdown of each part of the workflow.


.. include:: _static/signe_run.yml
   :literaL:


Detailed explanation of the workflow
------------------------------------

* apiVersion: Specifies the API version for Argo workflows (argoproj.io/v1alpha1).
* kind: Indicates the type of Kubernetes resource, which is Workflow in this case.
* metadata: Contains metadata about the workflow.
      * generateName: A prefix for the name of the workflow.
* spec: Defines the specification of the workflow.
   * entrypoint: Specifies the entrypoint template for the workflow. The entrypoint is the main workflow template that will be executed first.
* templates: Defines the list of templates used in the workflow.
    * container: Specifies the container to run.
        * image: The Docker image to use.
        * command: The command to execute inside the container (python ./StemCloud/example/example_stem.py).
    * outputs: Specifies the outputs of the template.
        * artifacts: Defines the artifacts to be captured from the container.
            * name: The name of the artifact.
            * path: The path inside the container where the artifact is located (/uvec_train_model).
            * s3: Specifies the S3 bucket details for storing the artifact.
                * endpoint: The S3 endpoint URL (s3.amazonaws.com).
                * bucket: The name of the S3 bucket (stem-data).
                * key: The S3 object key prefix (data).
                * region: The AWS region for the S3 bucket (eu-west-1).
                * accessKeySecret and secretKeySecret: Specifies the Kubernetes secrets used for S3 access.
                    * name: The name of the secret (my-s3-credentials).
                    * key: The specific key within the secret (accessKey and secretKey).
            * archive: Specifies how the artifact should be archived.
                * none: Indicates no archiving is required.

The following is an example of a workflow that can be used to run multiple STEM calculations in parallel using Argo.

.. include:: _static/multiple_runs.yml
   :literaL:


In this case the workflow can perform parallel calculations by running multiple instances of the same Python script in parallel.
This is achieved and defined in the generate-range template, which generates a list of numbers from 1 to 9 (in this case).
If you want ude the generate-range step to create inputs from the signe-step-run , you can adjuct the command and arguments section.
The workflow will be modified as follows:


.. code-block:: yaml

    - name: signe-run-step
      inputs:
        parameters:
        - name: range
        - name: additional-param
          value: "some_value"
      nodeSelector:
        alpha.eksctl.io/nodegroup-name: c7ixlarge
      container:
        image: containers.*********/stem/stem:latest 
        command: [python, ./StemCloud/example/example_stem.py, "{{inputs.parameters.range}}", "{{inputs.parameters.additional-param}}"]


In this example, an additional parameter additional-param is added, and it is included in the command executed by the signe-run-step template. 
This allows the script to use more complex inputs.