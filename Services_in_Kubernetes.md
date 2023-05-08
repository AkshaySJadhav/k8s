
## NodePort configuration for HTTPD Deployment.

#### 1. Creating deployment.
```
apple@Apples-MacBook-Pro ~ % kubectl create deployment webhosting --image=httpd
deployment.apps/webhosting created

apple@Apples-MacBook-Pro ~ % kubectl get deployment.apps/webhosting
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
webhosting   1/1     1            1           18s
```

#### 2. Create a NodePort service on 30007 port.

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

#### 3. Getting the Node IP address to access the http page.


```
apple@Apples-MacBook-Pro ~ % kubectl get nodes -owide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   118m   v1.26.3   192.168.121.2   <none>        Ubuntu 20.04.5 LTS   5.15.49-linuxkit   docker://23.0.2
apple@Apples-MacBook-Pro ~ %
````


```
apple@Apples-MacBook-Pro ~ % curl 192.168.121.2:30007
<html><body><h1>It works!</h1></body></html>
```
