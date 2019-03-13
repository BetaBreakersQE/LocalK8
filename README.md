# Jenkins Integration using Kubernetes for UNIX systems

Using Jenkins configuration as code via Kubernetes. It will allow you to run the same particular instance of Jenkins.

Follow these steps correctly and Jenkins will be launched on your system
## Tech-Stack Used
[VirtualBox](https://www.virtualbox.org/wiki/Downloads)
[Minikube](https://kubernetes.io/docs/setup/minikube/)
[Docker](https://docs.docker.com/docker-for-mac/install/)
[Hamachi](https://www.vpn.net/) is optional if you are not on the same LAN. 
The rest will be configured inside your machine via terminal.

# Setup
## Pre-Requisite: Installing VirtualBox
Make sure to have an updated version of [VirtualBox](https://www.virtualbox.org/wiki/Downloads) in order to install Minikube
## Installing and Starting Minikube

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
## Installing and Starting Kubernetes

You will need to download kubernetes and it will be configured to minikube.

```bash
brew install kubernetes-cli
```

To ensure that kubernetes is installed correctly:

```bash
kubectl version
```

## Installing Docker & Building the Dockerfile

Setup and have [Docker](https://www.docker.com/get-started) downloaded on your system
OSX & Unix: 

Verify that it is working:

```bash
docker version
```

Also run this command and make sure it does NOT show any errors:

```bash
eval $(minikube docker-env)
```
Now make a Dockerfile if you do not have one and put all of the necessary parameters needed to build the docker image. You will need to go into the directory that contains the Dockerfile

```bash
docker build -t <your_docker_username>/<name_of_jenkins_image> .
```
## Configuring and building YAML files
Now build a deployment YAML file for the configuration parameters 
Here is what mine looks like.

```bash
vi jenkins-deployment.yaml
```
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins
          image: ybushnev/my-jenkins-image:1.0
          env:
            - name: JAVA_OPTS
              value: -Djenkins.install.runSetupWizard=false
          ports:
            - name: http-port
              containerPort: 8080
            - name: jnlp-port
              containerPort: 50000
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-home
          emptyDir: {}
```
Next we will apply the yaml file.

```bash
kubectl apply -f jenkins-deployment.yaml
```

It should display:

```bash
deployment "jenkins" created
```

Now verify jenkins is there using kubectl:

```bash
kubectl describe pod | grep jenkins
```

Now create the service yaml file where we need to specify the ports and necessary connection needed. Jenkins should maintain connection to the pod while the IP changes.

Here is an example of my jenkins-service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app: jenkins
```

Run it inside the Kubernetes container:

```bash
kubectl create -f jenkins-service.yaml
```

It should return:

```bash
service "jenkins" created
```

Open the dashboard and make sure everything is running properly.

```bash
minikube dashboard
```

## Starting Jenkins

Once everything is working, start Jenkins

```bash
kubectl get service
```

Now go over to the output and look for the line that says

```bash
<jenkins> NodePort <ip_address> 8080:<port_number>/TCP
```

Also get the minikube ip

```bash
minikube ip
```

Now go to your browser and type in http://minikubeip:port_number from above

That will be your Jenkins instance

## Password
We will have to get the initial admin password in a different manner. Run this command:

```bash
kubectl get secret my-release-jenkins -o yaml
```
Go over and find the value of jenkins-admin-password. And then do:

```bash
#include the ' '
echo '<jenkins-admin-password_value>' | base64 --decode
```
It will output your password and the username will be admin.

## Configuring CI/CD & Pipeline

Navigate to Manage Jenkins > Manage Plugins

Install Configuration as Code and Pipeline (if not present from initial installation). 

Feel free to grab whatever plugins you need under Manage Plugin

Create a new item and create a new pipeline. Name your pipeline, then go to Configure. Then go down to the pipeline script from SCM option. Enter the link to your github repo, and under credentials enter your github credentials. For Script Path parameter it should be 'Jenkinsfile'

Create a Jenkinsfile inside the repo if you haven't. Now save and run it.

My default Jenkinsfile is in the repo.

```bash
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
## Slave 

Navigate to manage Jenkins > manage node > new node. Then provide the name of the node and categorize it as a permananent agent.

Now for the root directory, specify the directory that you want to work in. Then launch the node via ssh. 

The option 'launch agent agents via ssh' should be selected. 
The host is the IP address to the network. 
And the credentials should be the user to the hostname and the sudo password. The host key should be 'non verifying verification strategy'.
 Also keep the slave node online as much as possible.

In the slave configuration, go and specify the IP address that the Hamachi server provides. (Pretty much ssh into there).

### Running in Virtual Machine

Find the IP address of your Virtual Machine.

```bash
ifconfig -a
```
To test it, ping it:

```bash
ping <ipaddress>
```

Then ssh into VM

```bash
ssh <ipaddress>
```

## Amazon Web Services

In progress...
