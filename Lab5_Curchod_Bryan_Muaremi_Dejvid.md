<div style="text-align: justify">  


# CLD Labo 05 : KUBERNETES

> Authors : Curchod Bryan - Muaremi Dejvid    
> Professor : Graf Marcel   
> Assistants : Gardel Bastian   
> Date : 18.04.2019  


## Task 1 - Deploy the application on a local test cluster

In this task you will set up a local Kubernetes test cluster and deploy an example application to it.



The goal of the provided example application is to store and edit a To Do list. The application consists of a simple Web UI (Frontend) that uses a REST API (API service) to access a Key-Value storage service (Redis). Ports to be used for Pods and Services are shown in the figure.

The required Docker images are provided on Docker Hub:

Frontend: icclabcna/ccp2-k8s-todo-frontend
API backend: icclabcna/ccp2-k8s-todo-api
Redis: redis:3.2.10-alpine
SUBTASK 1.1 - INSTALLATION OF MINIKUBE
Minikube is a tool that makes it easy to run Kubernetes locally. Minikube creates a single-node Kubernetes cluster inside a virtual machine on your local machine.

Minikube requires a hypervisor to run the virtual machine, such as VirtualBox, VMware, KVM, Hyper-V or xhyve. We recommend VirtualBox.

Install Minikube. It is currently distributed as binary for macOS, Linux and Windows. Download the binary, then make it executable (macOS and Linux). If you want you can add it to the path.

macOS:

$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.22.0/minikube-darwin-amd64
Linux:

$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.22.0/minikube-linux-amd64
Windows (using MSYS bash or Windows Subsystem for Linux):

$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.22.0/minikube-windows-amd64.exe
On Windows use minikube.exe instead of just minikube when executing a command.

Verify successful installation by running

$ minikube version
minikube version: v0.22.0
If you have trouble installing, consult the installation instructions for Minikube v0.22.0.

SUBTASK 1.2 - INSTALLATION OF KUBECTL
Install kubectl. If you have installed the Google Cloud SDK you already have kubectl (in this case do a gcloud components update to get the latest version). Kubectl is currently distributed as binary for macOS, Linux and Windows. Download the binary, then make it executable (macOS and Linux). If you want you can add it to the path.

macOS:

$ curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/v1.8.6/bin/darwin/amd64/kubectl
Linux:

$ curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/v1.8.6/bin/linux/amd64/kubectl
Windows:

$ curl -Lo kubectl.exe http://storage.googleapis.com/kubernetes-release/release/v1.8.6/bin/windows/amd64/kubectl
On Windows use kubectl.exe instead of just kubectl when executing a command.

Verify successful installation by running

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.6", [...]
SUBTASK 1.3 - CREATE A ONE-NODE CLUSTER ON YOUR LOCAL MACHINE
Create and start a one-node cluster. Run

$ ./minikube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Downloading Minikube ISO
 150.53 MB / 150.53 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Downloading kubeadm v1.10.0
Downloading kubelet v1.10.0
Finished Downloading kubelet v1.10.0
Finished Downloading kubeadm v1.10.0
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
This will download a VM image for the Kubernetes cluster and start it. Also the tool kubectl is configured to use this cluster by default.

Examine the cluster:

$ kubectl cluster-info
You should see a running Kubernetes master.

Note that the displayed URLs will not work in your browser as authentication is not configured. We will instead use kubectl proxy (see below).

To view the nodes in the cluster, run the kubectl get nodes command:

$ kubectl get nodes
This command shows all nodes that can be used to host our applications. Now we have only one node, and we can see that its status is Ready (it is ready to accept applications for deployment).

Detailed info on the available kubectl commands, syntax and parameters can be found in the kubectl cheat sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

OPTIONAL: ACCESS THE KUBERNETES DASHBOARD
The Kubernetes Dashboard gives you a graphical web interface to your Kubernetes cluster. It is itself bundled as a Pod, therefore it runs on the cluster as any other Pod (although in a separate namespace). The Dashboard is already included with Minikube.

To show graphs of resource usage, such as CPU and memory, Dashboard relies on a metrics collecting tool, such as Heapster. Again, Heapster is bundled as a Pod.

To install Heapster, run:

$ kubectl apply --filename https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
$ kubectl apply --filename https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/standalone/heapster-controller.yaml
To access Dashboard, run:

$ minikube dashboard
This will open Dashboard in the default browser.

SUBTASK 1.4 - DEPLOY THE APPLICATION
Next you create the configuration and deploy the three tiers of the application to the Kubernetes cluster.

DEPLOY THE REDIS SERVICE AND POD
Use the following commands to deploy Redis using the provided configuration files:

kubectl create -f redis-svc.yaml

kubectl create -f redis-pod.yaml
and verify it is up and running and on which ports using the command kubectl get all.

To zoom in on a Kubernetes object and see much more detail try kubectl describe po/redis for the Pod and kubectl describe svc/redis-svc for the Service.

DEPLOY THE TODO-API SERVICE AND POD
Using the redis-svc.yaml file as example and information from api-pod.yaml, create the api-svc.yaml configuration file for the API Service. The Service has to expose port 8081 and connect to the port of the API Pod.

Be careful with the indentation of the YAML files. If your code editor has a YAML mode, enable it.

Deploy and verify the API-Service and Pod (similar to the Redis ones) and verify that they are up and running on the correct ports.
DEPLOY THE FRONTEND POD
Using the api-pod.yaml file as an example, create the frontend-pod.yaml configuration file that starts the UI Docker container in a Pod.

Docker image for frontend container on Docker Hub is icclabcna/ccp2-k8s-todo-frontend
Note that the container runs on port: 8080

It also needs to be initialized with the following environment variables (check how api-pod.yaml defines environment variables):

API_ENDPOINT_URL: URL where the API can be accessed e.g., http://localhost:9000
What value must be set for this URL?
Hint: remember that anything you define as a Service will be assigned a DOMAIN that is visible via DNS everywhere in the cluster and a PORT.

Deploy the Pod using kubectl.
VERIFY THE TODO APPLICATION
Now you can verify the the ToDo application is working correctly by connecting to the Frontend Pod. As the Pod's IP address is only accessible from inside the cluster, we have to use a trick.

The kubectl program is able to establish a secure tunnel between your local machine and the cluster. It listens on a port on your local machine and as soon as someone (e.g., your browser) establishes a TCP connection to that port, kubectl forwards the connection to the cluster. This is called port forwarding. There is one limitation, though: the TCP connection can only be forwarded to a Pod.

To start port forwarding, run

$ kubectl port-forward pod_name local_port:pod_port
where pod_name is the name of the Frontend Pod, pod_port is the port where the Frontend Pod is listening, and local_port is a free port on your local machine, say 8001.

The command blocks and keeps running to keep the tunnel open. Using your browser or another terminal window you can now access the Pod at http://localhost:local_port.

You should see the application's main page titled Todos V2 and you should be able to create a new To Do item. Be patient, the application will be very slow at first.

DELIVERABLES
Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

TROUBLESHOOTING
Several things can be misconfigured. Remember that there are two Service dependencies:

the Frontend forwarding requests to the (not externally accessible) API Service;
the API Service accessing the Redis Service (also only accessible from within the cluster).
CONSULTING THE POD LOGS
You can look at the Pod logs to find out where the problem is. You can see the logs of a Pod using:

$ kubectl logs -f pod_name
CONNECTING TO A POD
You may want to test if a Pod is responding correctly on its port. Use kubectl port-forward as described at the end of task 1.

RUNNING A COMMAND IN A CONTAINER
A handy way to debug is to log in to a container and see whether the required services are visible to the container.

You can run a command on a (container in a) Pod using:

$ kubectl exec -it pod_name <command>
E.g. use kubectl exec -it pod-name bash to start a Bash shell inside the container.

Container images tend to contain only the strict minimum to run the application. If you are missing a tool such as curl you can install it (assuming a Debian-family distribution):

$ apt-get update
$ apt-get install curl
TASK 2 - DEPLOY THE APPLICATION IN KUBERNETES ENGINE
In this task you will deploy the application in the public cloud service Google Kubernetes Engine (GKE).

SUBTASK 2.1 - CREATE PROJECT
Log in to the Google Cloud console at http://console.cloud.google.com/. Navigate to the Resource Manager (https://console.cloud.google.com/cloud-resource-manager>) and create a new project.

SUBTASK 2.2 - CREATE A CLUSTER
Go to Google Kubernetes Engine (GKE) (https://console.cloud.google.com/kubernetes/), then create a cluster. Give it a name of the form gke-cluster-1, select a zone close to you and set the Size to 2. Keep the other settings at their default values.

SUBTASK 2.3 - DEPLOY THE APPLICATION ON THE CLUSTER
Once the cluster is created, the GKE console will show a Connect button next to the cluster in the cluster list. Click on it. A dialog will appear with a command-line command. Copy/paste the command and execute it on your local machine. This will download the configuration info of the cluster to your local machine (this is known as a context). It also changes the current context of your kubectl tool to the new cluster.

To see the available contexts, type

$ kubectl config get-contexts
You should see two contexts, one for the Minikube cluster and one for the GKE cluster. The current context has a star * in front of it. The kubectl commands that you type from now on will go to the cluster of the current context.

With that you can use kubectl to manage your GKE cluster just as you did in task 1. Repeat the application deployment steps of task 1 on your GKE cluster.

Should you want to switch contexts, use

$ kubectl config use-context <context>
SUBTASK 2.4 - DEPLOY THE TODO-FRONTEND SERVICE
On the Minikube cluster we did not have the possibility to expose a service on an external port, that is why we did not create a Service for the Frontend. Now, with the GKE cluster, we are able to do that.

Using the redis-svc.yaml file as an example, create the frontend-svc.yaml configuration file for the Frontend Service.

Unlike the Redis and API Services the Frontend needs to be accessible from outside the Kubernetes cluster as a regular web server on port 80.

We need to change a configuration parameter. Our cluster runs on the GKE cloud and we want to use a GKE load balancer to expose our service.
Read the section "Publishing Services - Service types" of the K8s documentation https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types
Deploy the Service using kubectl.
This will trigger the creation of a load balancer on GKE. This might take some minutes. You can monitor the creation of the load balancer using kubectl describe.

VERIFY THE TODO APPLICATION
Now you can verify if the ToDo application is working correctly.

Find out the public URL of the Frontend Service load balancer using kubectl describe.
Access the public URL of the Service with a browser. You should be able to access the complete application and create a new ToDo.
DELIVERABLES
Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report (if they are unchanged from the previous task just say so).

Take a screenshot of the cluster details from the GKE console. Copy the output of the kubectl describe command to describe your load balancer once completely initialized.

TASK 3 - ADD AND EXERCISE RESILIENCE
By now you should have understood the general principle of configuring, running and accessing applications in Kubernetes. However, the above application has no support for resilience. If a container (resp. Pod) dies, it stops working. Next, we add some resilience to the application.

SUBTASK 3.1 - ADD DEPLOYMENTS
In this task you will create Deployments that will spawn Replica Sets as health-management components.

Converting a Pod to be managed by a Deployment is quite simple.

Have a look at an example of a Deployment described here: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
Note that the doc assumes that the Kubernetes server supports the API version apps/v1. But the Kubernetes version used by Google supports only apps/v1beta1 (you can check with kubectl api-version). Your file thus needs to start with: apiVersion: apps/v1beta1
Create Deployment versions of your application configurations (e.g. redis-deploy.yaml instead of redis-pod.yaml) and modify/extend them to contain the required Deployment parameters.
Again, be careful with the YAML indentation!
Make sure to have always 2 instances of the API and Frontend running.
Use only 1 instance for the Redis-Server. Why?
Delete all application Pods (using kubectl delete pod ...) and replace them with deployment versions.
Verify that the application is still working and the Replica Sets are in place. (kubectl get all, kubectl get pods, kubectl describe ...)
SUBTASK 3.2 - VERIFY THE FUNCTIONALITY OF THE REPLICA SETS
In this subtask you will intentionally kill (delete) Pods and verify that the application keeps working and the Replica Set is doing its task.

Hint: You can monitor the status of a resource by adding the --watch option to the get command. To watch a single resource:

kubectl get <resource-name> --watch
To watch all resources of a certain type, for example all Pods:

kubectl get pods --watch
You may also use kubectl get all repeatedly to see a list of all resources. You should also verify if the application stays available by continuously reloading your browser window.

What happens if you delete a Frontend or API Pod? How long does it take for the system to react?
What happens when you delete the Redis Pod?
How can you change the number of instances temporarily to 3? Hint: look for scaling in the deployment documentation
What autoscaling features are available? Which metrics are used?
How can you update a component? (see "Updating a Deployment" in the deployment documentation)
SUBTASK 3.3 (OPTIONAL) - PUT AUTOSCALING IN PLACE AND LOAD-TEST IT
On the GKE cluster deploy autoscaling on the Frontend with a target CPU utilization of 30% and number of replicas between 1 and 4. Load-test using JMeter.

DELIVERABLES
Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

CLEANUP
At the end of the lab session:

Delete the GKE cluster
Delete the project
If you want to remove the Minikube Virtual Machine from your local machine, run

$ minikube stop
$ minikube delete
