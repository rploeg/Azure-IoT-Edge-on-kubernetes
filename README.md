# Install and running Microsoft Azure IoT Edge on a kubernetes cluster on Docker for Windows

## Goal

In this tutorial I like to explain how to install a kubernetes cluster for Azure IoT Edge on Docker. The current issues with Azure IoT Edge that it not have a official fail over system. This is the first option, that is supported by Microsoft, that we have now. This is still in public preview (June 2019)

## Architecture of Azure IoT edge on a cluster

Below you see the architecture of Azure IoT Edge on a kubernetes cluster. (image from [Microsoft](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-kubernetes))
![Architecture](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/k8s-arch.png)

## Installation

### Install Docker and enable Kubernetes
First of all install Docker Desktop for Windows on your machine ([download](https://docs.docker.com/docker-for-windows/install/))

After the installation is complete, go to the settings of Docker for Windows and select Kubernetes:

![](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/SettingKubernetesDocker.png)

Click on Apply, this can take a while depending on speed of your machine. 

### Configure Kubernetes

By default enabling Kubernetes in the Docker for Windows settings will install the kubectl binary and add it to your environment variables.

To verify kubectl is installed open a terminal on your machine:

```
kubectl version
```
It should give a version number back. If not, the Kubernetes is not enabled. After that you have to change the context of Kubectl to the docker desktop cluster. Please use the following command to change that:

```
kubectl config use-context docker-for-desktop
```

### Optional - Kubernetes Dashboard

There is a great dashboard for kubernetes ([Link](https://github.com/kubernetes/dashboard)) that is usefull to have, you can install it by entering the following command in your terminal:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

To start the dashboard enter the following command and after that start your browser to see the dashboard:

```
kubectl proxy
```
If you want to bypass authentication, this is off course not recommended, you can follow [this](https://devblogs.microsoft.com/premier-developer/bypassing-authentication-for-the-local-kubernetes-cluster-dashboard/) guide. 

### Install Helm - package manager for Kubernetes

For installing the IoT Edge on kubernetes you first need to install a package manager called Helm. There is a great documentation [here](https://helm.sh/docs/using_helm/#quickstart-guide) how you can install it. 


### Create Azure IoT Edge device in the Azure Portal

Now it's time to create a new Azure IoT Edge Device. You can do this to navigate to the user portal, select the IoT Hub you already have (or create a new one) and click on IoT Edge and create a new Device:

![IoTEdge](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/CreateAzureIoTEdge.png)

If you click on the newly created Azure IoT Edge Device you can now copy your connection string:

![ConnectionString](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/CopyConnectionString.png)

### Install Azure IoT Edge on kubernetes

First of all you need to initiate the Helm component:

```
helm init
```

Add the package of Azure IoT Edge to the repo

```
helm repo add edgek8s https://edgek8s.blob.core.windows.net/helm/
helm repo update
```

Get your connection string from the Azure IoT Edge and copy the string in the below command:

```
helm install \
--name k8s-edge1 \
--set "deviceConnectionString=PUTHEREYOURCONNECTIONSTRING" \
edgek8s/edge-kubernetes
```

### Check if installation is complete

Navigate to the Azure Portal and select the IoT hub and your IoT Edge device. The Edge Agent should be running:

![Running](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/runningcluster.png)


### Deploy a module
You can now deploy for example the Temperature Runtime Module, so simulate temperature values from your cluster. Go to the Azure Portal, click on new and search for the Simulated Temperature Sensor

![TempSensor](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/NewSimulateTempSensor.png)

Find your device and click on create and on the following screens hit the next, next and finish button:

![Conf](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/CreateSimulateTempSensor.png)

Wait for some minutes and you should see the new modules running on your cluster now:

![ClusterRunning](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/RunningModules.png)

### Check the logs of the container

If you want to check the containers (if something goes wrong for example), simply start the kubernetes dashboard, navigate to your container and select logs. In the below image you see the simulated temperature sensor messages that are send to the IoT Hub

![CheckMessage](https://github.com/rploeg/Azure-IoT-Edge-on-kubernetes/blob/master/CheckMessagesSend.png)

## Recap

In this tutorial you have seen how to install Azure IoT Edge on Kubernetes, it's quite simple and really powerfull in high end edge scenario's where you need failover if it's critical.

And if you want to delete the Azure IoT Edge from your cluster, please enter the following command:

```
helm delete --purge k8s-edge1
```
