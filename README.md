***Commands used in Basic Kubernetes TAM Bootcamp Session 1***

##Install kubectl 

In order to use the Kubernetes cluster, you need a CLI and hence you need kubectl to be installed from https://kubernetes.io/docs/tasks/tools/ 

On Mac: 
Using brew:

    brew install kubectl 

Check the version using:

    kubectl version --client

On Windows: 
Using chocolatey:

    choco install kubernetes-cli

Using curl: 

    curl -LO https://dl.k8s.io/release/v1.20.0/bin/windows/amd64/kubectl.exe

Check the version using:

    kubectl version --client

##Install KinD 

On Mac: 

    brew install kind

Windows: 

    curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.10.0/kind-windows-amd64
    Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe

You could also use chocolatey - https://chocolatey.org/packages/kind

    choco install kind 

Oh yes, if you havent installed Docker for Desktop, you should do that first since KinD is Kubernetes in Docker. 

You can create a Kubernetes cluster using  

    kind create cluster --name insert_name_of_your_cluster_here

    % kind create cluster --name vmwtam
    Creating cluster "vmwtam" ...
    ✓ Ensuring node image (kindest/node:v1.18.2) 🖼
    ✓ Preparing nodes 📦
    ✓ Writing configuration 📜
    ✓ Starting control-plane 🕹️
    ✓ Installing CNI 🔌
    ✓ Installing StorageClass 💾
    Set kubectl context to "kind-vmwtam"
    You can now use your cluster with:

    kubectl cluster-info --context kind-vmwtam

    Thanks for using kind! 😊

This will create a single node kubernetes cluster. Give it a minute for it to be ready :) 

For our labs, a single node cluster shoudl do just fine!

If you would like to create a multi-node cluster, you will have to create a config yaml file and pass that as a parameter when creating the k8s cluster. 

One example of a 3-node cluster is :


    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker

Save this file as a yaml file, I will call it multinode-config.yaml
Now create the kind cluster using the command: 

    kind create cluster --config multinode-config.yaml


The default location where the configuration is stored is 
${HOME}/.kube/config

PROTIP: 
It would be wise to increase the memory on your Docker Desktop. 





Alright now with kubectl installed and with a kind k8s cluster, let's start exploring it.

1> Create a Namespace

    kubectl create ns bootcamp

2> List the namespaces

    kubectl get ns

3> Create a nginx pod in the bootcamp namespace

    kubectl run mynginx --image=nginx -n bootcamp

4> Check the pods in the bootcamp namespace

    kubectl get pods -n bootcamp

5> See the details of the nginx pod using the command: 

    kubectl describe pod mynginx -n bootcamp

6> Now lets create an nginx deployment with 2 replicas.

    kubectl create deployment nginxdeployment --image=nginx  --replicas=2 -n bootcamp

7> Lets look at the deployments now:

    kubectl get deploy -n bootcamp

You would see that it would state Ready as 2/2 and available as 2. 

8> Deployments create pods in the background, let's take a look at how the pods look in the bootcamp namespace using the command: 

    kubectl get pods -n bootcamp

Notice the pod names have additional characters beyond the deployment name 

9> Let's check if the replicas concept is working, by deleting one of the deployment pods.

    kubectl -n bootcamp delete pod nginxdeployment-5878566b7d-gdnt8

list all the pods again using 

    kubectl get pods -n bootcamp 

 you will see that there is another pod which is created since we specified 2 replicas in the deployment. So the replicas concept does work indeed!

10> You can delete the mynginx pod using: 

    kubectl -n bootcamp delete pod mynginx

If you do this, you can run the command in 3> again.

11> Now that we created pods and deployments, let's expose that mynginx pod using a service 

    kubectl -n bootcamp expose pod mynginx --name nginxsvc --port 80

12> The service that is created is of the type ClusterIP, which means that the service is reachable from within the cluster, now to access the mynginx pod, we can create a sort of toolbox pod where we could run our curl commands in it to check basic connectivity. 

    kubectl -n bootcamp run my-shell -it --image ubuntu -- /bin/bash

Once that pod gets deployed, you will see your shell prompt getting changed. You are now in the ubuntu pod. This pod wont have any utilities installed so lets install them using apt. 

    apt update
    apt install curl dnsutils iputils-ping

13> Open a new terminal tab or another terminal and get the name of the service using: 

    kubectl get svc -A

14> Note the name of the service and get back to the terminal tab where you had the ubuntu pod in step 12.

 We can now do a curl 

    curl nginxsvc

You should see the Welcome to nginx! message. 

*A note on name resolution, services within kubernetes have a FQDN of the syntax*

    <service_name>.<namespace_name>.svc.cluster.local

15> You can ping the pod from the ubuntu pod , but first you will have to find the IP address of the mynginx pod, lets do that by:

    kubectl get pods -A -o wide
Note the IP of the mynginx pod and now you could ping that IP address using

    ping IP_ADDR_OF_MYNGINX_POD

16> If you had an issue with a pod or a deployment , you could take a look at the logs using the kubectl logs command

    kubectl -n bootcamp logs mynginx

17> You can also do a port-forward on the mynginx pod to access it from your browser on localhost using: 

    kubectl -n bootcamp port-forward mynginx 8080:80

Now open a browser and point to locahost:8080 to see the Welcome to nginx page. 

18> DaemonSets: An easy way to create a DaemonSet is to first create a deployment using the --dry-run -o yaml option and then editing fields in there : 

PRO-TIP : I use --dry-run quite a lot using imperative commands so that I dont have to write the yamls ;) 

Create a deployment using 

    kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml

Here our objective is to create an elasticsearch daemonset. 

Now edit fluentd.yaml using vi or anything and do the following: 
1. Change kind: Deployment to kind:DaemonSet
2. Remove the replicas:1 line
3. Remove the status:{} line
4. Remove the resources:{} line.
5. Remove the strategy{} line.  

Save the file and now apply it using kubectl apply

    kubectl apply -f fluentd.yaml

Verify if you have the DaemonSet created using 

    kubectl get ds

19> A note about contexts in kubernetes, when you are running multiple K8S clusters and have access to it. You can see the contexts using 

    kubectl config get-contexts

You can set a context using 

    kubectl config set-context NAME_OF_CONTEXT

20> Looking at the type of resources and versions of APIs available, also shortnames. 

    kubectl api-resources
    kubectl api-versions
