## Kubernetes on Mesos ##

This tutorial will walk you through setting up Kubernetes on a Mesosphere cluster. It provides a step by step walk through of adding Kubernetes to a Mesosphere cluster and running the classic nginx demo application.

### Introduction of Kubernetes and mesos ###

You can get the detail information of Kubernetes and mesos from the following pages.

[Kubernetes](http://yeasy.gitbooks.io/docker_practice/content/kubernetes/index.html)

[Mesos-English](http://cloudarchitectmusings.com/2015/03/23/apache-mesos-the-true-os-for-the-software-defined-data-center/)

[Mesos-Chinese](http://www.infoq.com/cn/articles/analyse-mesos-part-02?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk)

###Prerequisites###

- Understanding of Apache Mesos
- A Running Mesosphere cluster
- A machine in the cluster which should become the Kubernetes master node with:
  * GoLang > 1.2
  * make (i.e. build-essential)
  * Docke

### Deploy Kubernetes-Mesos ###

Log into the master node over SSH.
Build the Kubernetes-Mesos executables.
    
    #dpkg -l|grep mesos
    #git clone https://github.com/mesosphere/kubernetes-mesos k8sm
    #cd k8sm/hack/images/dcos
    #make

  Kubernetes-Mesos executables km, kubectl, etcd and etcdctl were build in the folder _build.
   



### Validate KM Services ###

Add the appropriate binary folder to your `PATH` to access kubectl.Set some environment variables. The internal IP address of the master is visible via the cluster details page on the Mesosphere launchpad:

    #export PATH=k8sm/hack/images/dcos/_build:$PATH
    #export servicehost=${mesos_master_internal_ip_address}
    #export KUBERNETES_MASTER=http://${servicehost}:8888

Interact with the kubernetes-mesos framework via `kubectl`:


    #kubectl get services
> NAME  LABELS    SELECTOR   IP(S)          PORT(S)
> 
> k8sm-scheduler   component=scheduler,provider=k8sm         <none    10.10.10.113   10251/TCP
> 
> kubernetes      component=apiserver,provider=kubernetes   <none    10.10.10.1     443/TCP
```

Lastly, look for Kubernetes in the Mesos web GUI by pointing your browser to `http://<mesos-master-ip:port>`. Go to the Frameworks tab, and look for an active framework named "Kubernetes".


### Spin up a pod ###

Write a JSON pod description to a local file:

    #cat <<EOPOD >nginx.yaml
    yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      name: nginx
    image: nginx  
    ports:
      containerPort: 80
    EOPOD
Send the pod description to Kubernetes using the `kubectl` CLI:


    #kubectl create -f ./nginx.yaml pods/nginx


Wait a minute or two while `dockerd` downloads the image layers from the internet.We can use the `kubectl` interface to monitor the status of our pod:

    #kubectl get pods

> NAME      READY     STATUS    RESTARTS   AGE
> 
> nginx     1/1       Running   0          14s


Verify that the pod task is running in the Mesos web GUI. Click on the
Kubernetes framework. The next screen should show the running Mesos task that started the Kubernetes pod.