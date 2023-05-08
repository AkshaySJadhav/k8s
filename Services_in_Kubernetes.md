
# Services in Kubernetes.


In Kubernetes, Pods are non-permanent resources - they can appear and disappear as needed. This is because Kubernetes constantly checks to make sure the cluster is running the desired number of replicas (copies) of your app. And Pods are created or destroyed to match this desired state.

Services provide discovery and routing between pods. For example, services connect an application front-end to its backend, each of which running in separate deployments in a cluster.


What are the types of Kubernetes services?
<br>
[1]. ClusterIP:- Exposes a service which is only accessible from within the cluster. <br>
[2]. NodePort:- Exposes a service via a static port on each node’s IP. <br>
[3]. LoadBalancer:- Exposes the service via the cloud provider’s load balancer.


## Configurure ClusterIP service for HTTPd containers/pod.


=> Create the deployment:
```
controlplane $ kubectl create deployment webhosting --image=httpd
deployment.apps/webhosting created
controlplane $
````

=> Creating the ClusterIP service for the application to expose the port at cluster level. While creating the clusterIP, make sure that you mentioned the pod label in selector section.
```
controlplane $ cat service.yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: my-clusterip-service
  name: my-clusterip-service
spec:
  ports:
  -  port: 8080
     targetPort: 80
  selector:
    app: webhosting
status:
  loadBalancer: {}
controlplane $ 

controlplane $ kubectl apply -f service.yaml 
service/my-clusterip-service created
controlplane $ 
```

This is how the service configuration looks like:
```
controlplane $ kubectl describe services                    
Name:              my-nodeport-service
Namespace:         default
Labels:            app=my-nodeport-service
Annotations:       <none>
Selector:          app=webhosting
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.106.15.24
IPs:               10.106.15.24
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         192.168.1.3:80
Session Affinity:  None
Events:            <none>
controlplane $ 
```

Kubernetes automatically creates the endpoint in the backed when service is created.
  
```
controlplane $ kubectl get endpoints
NAME                  ENDPOINTS         AGE
kubernetes            172.30.1.2:6443   6d19h
my-nodeport-service   192.168.1.3:80    2m15s
controlplane $ 
```

The webpage of the httpd service is accessible with the ClusterIP service IP address and the port.
```
  
controlplane $ curl -v 10.106.15.24:8080
<html><body><h1>It works!</h1></body></html>
 controlplane $  
 ```

## Configurure NodePort service for HTTPd containers/pod.


==> Creating deployment.
```
apple@Apples-MacBook-Pro ~ % kubectl create deployment webhosting --image=httpd
deployment.apps/webhosting created

apple@Apples-MacBook-Pro ~ % kubectl get deployment.apps/webhosting
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
webhosting   1/1     1            1           18s
```

==> Create a NodePort service on 30007 port.

```
apple@Apples-MacBook-Pro ~ % cat my-clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: my-nodeport-service
  name: my-nodeport-service
spec:
  ports:
  -  port: 8080
     targetPort: 80
     nodePort: 30007
  selector:
    app: webhosting
  type: NodePort
status:
  loadBalancer: {}


apple@Apples-MacBook-Pro ~ % kubectl apply -f my-clusterip-service.yaml
service/my-nodeport-service created

```

==> Getting the Node IP address to access to webpage on NodePort port=30007.
```
apple@Apples-MacBook-Pro ~ % kubectl get nodes -owide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   118m   v1.26.3   192.168.121.2   <none>        Ubuntu 20.04.5 LTS   5.15.49-linuxkit   docker://23.0.2
apple@Apples-MacBook-Pro ~ %

apple@Apples-MacBook-Pro ~ % curl 192.168.121.2:30007
<html><body><h1>It works!</h1></body></html>
```

## Configurure LoadBalancer for HTTPd containers/pod.

