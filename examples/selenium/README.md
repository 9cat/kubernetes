<!-- BEGIN MUNGE: UNVERSIONED_WARNING -->

<!-- BEGIN STRIP_FOR_RELEASE -->

<img src="http://kubernetes.io/img/warning.png" alt="WARNING"
     width="25" height="25">
<img src="http://kubernetes.io/img/warning.png" alt="WARNING"
     width="25" height="25">
<img src="http://kubernetes.io/img/warning.png" alt="WARNING"
     width="25" height="25">
<img src="http://kubernetes.io/img/warning.png" alt="WARNING"
     width="25" height="25">
<img src="http://kubernetes.io/img/warning.png" alt="WARNING"
     width="25" height="25">

<h2>PLEASE NOTE: This document applies to the HEAD of the source tree</h2>

If you are using a released version of Kubernetes, you should
refer to the docs that go with that version.

<strong>
The latest 1.0.x release of this document can be found
[here](http://releases.k8s.io/release-1.0/examples/redis/README.md).
Documentation for other releases can be found at
[releases.k8s.io](http://releases.k8s.io).
</strong>
--

<!-- END STRIP_FOR_RELEASE -->

<!-- END MUNGE: UNVERSIONED_WARNING -->

## Selenium on Kubernetes

Selenium is a browser automation tool used primarily for testing web applications. However when Selenium is used in a CI pipeline to test applications, there is often contention around the use of Selenium resources. This example shows you how to deploy Selenium to Kubernetes.

### Prerequisites
This example assumes you have a working Kubernetes cluster and a properly configured kubectl client. See the [Getting Started Guides](../../docs/getting-started-guides/) for details. 

Google Container Engine is also a quick way to get Kubernetes up and running: https://cloud.google.com/container-engine/

### Deploy Seleniumb Grid Hub:
We will be using Selenium Grid Hub to make our Selenium install scalable via a master/worker model. The Selenium Hub is the master, and the Selenium Nodes are the workers(not to be confused with Kubernetes nodes). We only need one hub, but we're using a replication controller to ensure that the hub is always running:
```
kubectl create -f selenium-hub-rc.yaml
```

The Selenium Nodes will need to know how to get to the Hub, let's create a service for the nodes to connect to.
```
kubectl create -f selenium-hub-svc.yaml
```

### Verify Selenium Hub Deployment
Let's verify our deployment of Selenium hub.

#### Kubernetes Nodes Reachable
If your Kubernetes nodes are reachable from your network, you can verify the hub like so:
```
NODEPORT=`kubectl get svc -l 'name=selenium-hub' -o template --template="{{ with index .items 0}}{{with index .spec.ports 0 }}{{.nodePort}}{{end}}{{end}}"`
NODE=`kubectl get nodes -o template --template="{{with index .items 0 }}{{.metadata.name}}{{end}}"`

curl http://$NODE:$NODEPORT
```

#### Kubernetes Nodes Unreachable
If you cannot reach your Kubernetes nodes from your network, you can proxy via kubectl.
```
export PODNAME=`kubectl get pods -l "name=selenium-hub" -o template --template="{{with index .items 0}}{{.metadata.name}}{{end}}"`
kubectl port-forward -p $PODNAME 4444:4444
```

In a seperate terminal, you can now check the status.
```
curl http://localhost:4444
```

#### Using Google Container Engine
If you are using Google Container Engine, you can expose your hub via the internet. This is a bad idea for many reasons, but you can do it as follows:
```
kubectl expose rc selenium-hub --name=selenium-hub-external --create-external-load-balancer=true
```

Then wait a few minutes, eventually your new `selenium-hub-external` service will be assigned an load balancing IP from gcloud.
```
get 2nd ip from kubectl.

curl
```

### Deploy Firefox and Chrome Nodes:
Now that the Hub is up, we can deploy workers.

This will deploy 3 Chrome nodes.
```
kubectl create -f selenium-node-chrome-rc.yaml
```

And 3 Firefox nodes to match.
```
kubectl create -f selenium-node-firefox-rc.yaml
```

Once the pods start, you will see them show up in the Selenium Hub interface.

### Run a Selenium Job
Let's run a quick Selenium job to validate our setup.

```

```

### Scale your Firefox and Chrome nodes.
If you need more Firefox or Chrome nodes, your hardware is the limit:
```
kubectl scale rc selenium-node-firefox --replicas=10
kubectl scale rc selenium-node-chrome --replicas=10
```

### Debugging
Sometimes it is neccessary to check on a hung test. Each pod is running VNC. To check on one of the browser nodes via VNC, it's reccomended that your proxy, since we don't want to expose a service to every node, and the containers have a weak password.
 
```
kubectl port-forward -p <POD_NAME> 9000:5900
```

Then connect to localhost:9000 with your VNC client using the password " "

Enjoy your scalable Selenium Grid!

Adapted from: https://github.com/SeleniumHQ/docker-selenium
