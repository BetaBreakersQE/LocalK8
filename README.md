# Jenkins Integration using Kubernetes for UNIX systems

Using Jenkins configuration as code via Kubernetes. It will allow you to run the same particular instance of Jenkins.

Follow these steps correctly and Jenkins will be launched on your system

##Pre-Requisite: Installing VirtualBox
Make sure to have an updated version of [VirtualBox](https://www.virtualbox.org/wiki/Downloads) in order to install Minikube
##Installing and Starting Minikube
First go ahead and retrieve Minikube, which is a tool that will allow you to run Kubernetes locally.

Use the following command to install Minikube

```bash
brew cask install minikube
```
To check that Minikube is installed, run the following command:

```bash
minikube version
```

Then to start minikube

```bash
minikube start
```
If VirtualBox and Minikube are both installed correctly, then you should see 

```bash
Done! Thank you for using minikube!
```
Now verify if it is running properly

```bash
minikube status
```
It should say something like 

```bash
 kubectl: Correctly configured: Pointing to minikube-vm at 192.169.99.101
```
##Installing and Starting Kubernetes

You will need to download kubernetes and it will be configured to minikube.

```bash
brew install kubernetes-cli
```

To ensure that kubernetes is installed correctly:

```bash
kubectl version
```

##Installing Docker & Building the Dockerfile


