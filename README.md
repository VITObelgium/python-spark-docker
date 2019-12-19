# Spark on Docker Sample
This code sample shows how to use Spark on Docker (http://spark.apache.org/) for distributed processing on the PROBA-V Mission Exploitation Platform. (https://proba-v-mep.esa.int/)
The sample intentionally implements a very simple computation:
for each PROBA-V tile in a given bounding box and time range, a histogram is computed. The results are then summed and printed. Computation of the histograms runs in parallel. For more details on how to run Spark without Docker, check out the Spark Quickstart Sample on [https://git.vito.be/projects/BIGGEO/repos/python-spark-quickstart/browse](https://git.vito.be/projects/BIGGEO/repos/python-spark-quickstart/browse)

## Docker for Driver and / or Executors
This sample application will run both the driver (appMaster) and the executors in a docker container. In this case both will use the same docker image, but this can be different docker images depending on your needs.
For the driver:
`--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_TYPE=docker`
For the executors:
`--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_TYPE=docker`

## Required Parameters
The 3 required parameters for running Spark on Docker are:
`YARN_CONTAINER_RUNTIME_TYPE=docker` For choosing to run on Docker
`YARN_CONTAINER_RUNTIME_DOCKER_IMAGE=your/image` To select which docker image you want to use
`YARN_CONTAINER_RUNTIME_DOCKER_MOUNTS=/var/lib/sss/pipes:/var/lib/sss/pipes:rw,/usr/hdp/current/:/usr/hdp/current/:ro,/etc/hadoop/conf/:/etc/hadoop/conf/:ro,/etc/krb5.conf:/etc/krb5.conf:ro` To set which folder you want to have mounted in your docker container. The ones mentioned here are always required for Spark on Docker to work.

This sample also needs the folder `/data/MTDA/TIFFDERIVED/PROBAV_L3_S1_TOC_333M` available in the executors, so this path is also added as read-only to the docker mounts (only of the executors).

## Py-files
Also the histogram python classes are added using spark-submit's '--py-files' parameter. These files will automatically be mounted in the container by YARN.

## Python
The docker container that we'll use has python3.6 installed with all the required python packages already preinstalled.
To use this python, set the `PYSPARK_PYTHON` environment variable before submitting, for example:
`export PYSPARK_PYTHON=/usr/bin/python3.6`

## Docker image
The Docker image used for your processing has certain requirements. It needs java 8, sssd client utilieties and a couple ENV variables set:`JAVA_HOME, HADOOP_HOME, HADOOP_CONF_DIR, YARN_CONF_DIR, SPARK_HOME`
We provide an image that has the necessary requirements set and is tested, called `vito-docker-private.artifactory.vgt.vito.be/dockerspark-centos`, which is in turn based on our own `vito-docker-private.artifactory.vgt.vito.be/centos7.4` image.

The docker image that was then created for this sample (vito-docker-private.artifactory.vgt.vito.be/dockerspark-quickstart) is using the following Dockerfile:

```
FROM vito-docker-private.artifactory.vgt.vito.be/dockerspark-centos

RUN yum install -y python36 python36-devel python36-pip && \
yum clean all

COPY lib/dockerspark-quickstart/pip.conf /etc/pip.conf

RUN /usr/bin/pip3 install --upgrade pip
RUN pip install catalogclient==1.2.0 rasterio==1.0.26 numpy==1.17.1
```
