[![Build Status](https://travis-ci.org/ibm-messaging/mq-docker.svg?branch=master)](https://travis-ci.org/ibm-messaging/mq-docker)

# Contents

This fork is a modified version of [the IBM repo](https://github.com/ibm-messaging/mq-docker), that showcases how to use the Salesforce Bridge. Checkout the original repo for information/instructions that go beyond this use case.

The IBM documentation about the Salesforce Bridge is available [here](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129300_.htm).

The steps to set up the enviroment are:
1. Build docker container
2. Create volume (required for Mac as host, recommended for other operating systems)
3. Start the Docker container
4. Add Verisign to JKS file in Docker container
5. Set up Salesforce connection in config file
6. Start Platform Event listener/publisher

The following pre-conditions in Salesforce need to be fulfilled:
- Creation of a Connected App
- Creation of a technical user incl. security token
- Creation of a self-signed certificate, exported as keystore

Theses elements will be used when setting up the bridge.


## Build

```bash
sudo docker build --tag mq .
```

## Create volume

Due to issues with MQ and the Mac file system you have to use a Docker volume instead of mounting the file system directly.

For this example a Docker volume with the name 'mq-data' needs to be created.

```bash
docker volume create mq-data
```

## Start the Docker image

```bash
docker run --env LICENSE=accept --env MQ_QMGR_NAME=QM1 --mount source=mq-data,target=/mnt/mqm --publish 1414:1414  --publish 9443:9443 --detach mq
```

## Add Verisign certificate to JKS file

Copy the self-signed certificate into the container. Add the signer certificates using the IBM Key Management command line.

```bash
runmqckm -cert -populate -db <yourkeystorefile>.jks
```

## Set up Salesforce connection

Follow the instructions outlined in step 7 of the [IBM Knowledge Center documentation](https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129310_.htm). The example values are good to go.

Make sure to enable _Subscribe to MQ publications for platform events?_ as outlined in step 7d.


# Example - Running the bridge for Platform Events

The [Dreamforce '18 Keynote demo](https://github.com/dreamforce17/purealoe) contains several platform events. Runing the following command from within the Docker container will listen for the event that gets fired when updating the Path compontent in a Product Bundle record.

```bash
/opt/mqm/bin/runmqsfb -f <your-mq-config>.cfg -r salesforce.log -e Bundle_Submitted -d1
```
