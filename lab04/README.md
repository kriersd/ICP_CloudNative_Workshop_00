
## Containerizing a process for deployment

**In this section**, you will learn how to upload a docker image Dockerhub and deploy it to our cluster. 

Development processes such as these are typically sped up with enhancements such as automated DevOps processes, however this is a helpful to get a fundamental look at how some of the underlying parts of the process work.

## What do I need to complete this lab?

* You need to have docker installed on your computer.

* You need a Dockerhub account. 
	*  https://hub.docker.com/signup
* You will need to have the kubectl command line installed on your computer.
	*  You did this in [lab 00](lab00/README.md)
	
***
**Note:** You will all be using a unique namespce, so please replace the namespace in the commands below with your unique namespace. You will be given your unique namespace at the beginning of this lab. 
***

## Task 1: Create a simple node.js process

Way back in [lab 02](lab02/README.md) we built an image with a node.js process in it and ran it in the Docker runtime on our local machines...remember that? Well now, we'll move a copy of that image to your Dockerhub account so its available to run in our IBM Cloud Private Cluster.

First lets tag our image. 

**Replace the ==Your-Docker-ID== with your own docker id**

```

$ docker tag mynode:v1.0 Your-Docker-ID/mynode:v1.0
$ docker images

```

"Tagging" the image tells it where to go when we "push" to a remote repository. By default it will upload to the repository at hub.docker.com (unless syou specify otherwise). 

Now we'll upload the image with "docker push":

**Replace the ==Your-Docker-ID== with your own docker id**

```
$ docker push irvingr/mynode:v1.0
```

From here, if you check your browser and look at the list of available tags in the repositoy we created, you should see your new image!

---

From here, we can run the image on our kubernetes cluster.

**Replace ==YOUR-UNIQUE-NAMESPACE== with the name space given to you by the instructor**

```
$ kubectl run hello-node --image=Your-Docker-ID/mynode:v1.0 --port=8080 --namespace=YOUR-UNIQUE-NAMESPACE
deployment "hello-node" created

$ kubectl get deployment --namespace=YOUR-UNIQUE-NAMESPACE
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           29s
$
$ kubectl get pods --namespace=YOUR-UNIQUE-NAMESPACE
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-77c96d485-qstmk   1/1       Running   0          3m

```

With the container running, we'll need to expose the applications port before we can access to application.

**Replace ==YOUR-UNIQUE-NAMESPACE== with the name space given to you by the instructor**

```
$ kubectl expose deployment hello-node --type=NodePort  --namespace=YOUR-UNIQUE-NAMESPACE
service "hello-node" exposed
$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   NodePort    10.97.250.220   <none>        8080:32644/TCP   4s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          5h

```

Now that the application is accessible via an external port, we're able to gain access to the services with curl

**Replace ==YOUR-UNIQUE-NAMESPACE== with the name space given to you by the instructor**

```  
$ kubectl get service --namespace=YOUR-UNIQUE-NAMESPACE
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   NodePort    10.97.250.220   <none>        8080:32644/TCP   1m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          5h


```
Then we can assemble the url as http:// + [icp master ip] + ":" + [port assigned by the service].
If the ip address for the master of our cluster is 192.168.99.100 then we should use http://192.168.99.100:30839 to access our new service.


We've built and run our first container on Kubernetes!

---
