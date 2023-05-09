# Playing with Network Policy.


If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster. NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network. NetworkPolicies apply to a connection with a pod on one or both ends, and are not relevant to other connections.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:
<br>
1. Pod Base Policy.
2. Namespace Base Policy.
3. Ipbased Policy.

We are going to create a policy that will deny all the communications of the pod for a specific namespace and we will improve the policy as per out requirement.

## Scenario 1: Deny All policy. 
---

- Creating the name space and creating the 2 pod's under same namespace.

```
controlplane $ kubectl create namespace development
namespace/development created
controlplane $ 

controlplane $ kubectl -n development run app-pod --image=nginx
pod/app-pod created
controlplane $ 

controlplane $ kubectl -n development run db-pod --image=nginx
pod/db-pod created
controlplane $ 


controlplane $ kubectl get pods --namespace=development
NAME      READY   STATUS    RESTARTS   AGE
app-pod   1/1     Running   0          14s
db-pod    1/1     Running   0          9s
controlplane $ 
controlplane $ 
```
- Now, let's expose the application port at the cluster level.

```
controlplane $ kubectl -n development expose pod app-pod --port=80
service/app-pod exposed
controlplane $ 
controlplane $ kubectl -n development expose pod db-pod --port=80
service/db-pod exposed
controlplane $ 

controlplane $ kubectl describe services -n development
Name:              app-pod
Namespace:         development
Labels:            run=app-pod
Annotations:       <none>
Selector:          run=app-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.183.250
IPs:               10.103.183.250
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.1.5:80
Session Affinity:  None
Events:            <none>


Name:              db-pod
Namespace:         development
Labels:            run=db-pod
Annotations:       <none>
Selector:          run=db-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.54.174
IPs:               10.100.54.174
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.1.6:80
Session Affinity:  None
Events:            <none>
controlplane $ 
```

- We do not have any Network policy at the moment hence pod app-pod is able to reach out to the db-pod and vice-versa.
  
```
controlplane $ kubectl -n development exec -it app-pod -- curl db-pod
<title>Welcome to nginx!</title>
controlplane $ 

controlplane $ kubectl -n development exec -it db-pod -- curl app-pod
<title>Welcome to nginx!</title>
controlplane $  
```

- Let's implement the networking policy that will block all communications.
  
```  
controlplane $ cat np.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: development
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
controlplane $ 


controlplane $ kubectl  apply -f np.yaml 
networkpolicy.networking.k8s.io/default-deny-all created
controlplane $ 
  
controlplane $ 
controlplane $ kubectl describe networkpolicy.networking.k8s.io/default-deny-all -n development
Name:         default-deny-all
Namespace:    development
Created on:   2023-05-09 07:16:46 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    <none> (Selected pods are isolated for ingress connectivity)
  Allowing egress traffic:
    <none> (Selected pods are isolated for egress connectivity)
  Policy Types: Ingress, Egress
controlplane $ 
```

- The nginx is not reachable which was reachable before implementing the policy.
      
```    
controlplane $ k exec -it app-pod -ndev -- curl db-pod
command terminated with exit code 130
controlplane $ 
      
controlplane $ k exec -it db-pod -ndev -- curl app-pod
command terminated with exit code 130
```

## Scenario 2: Let's setup the policy for both the pods so that app-pod will talk to db-pod.

- Creating the egress policy for app-pod to connect to db-pod.

```
controlplane $ cat allow-egress.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      run: app-pod
  policyTypes:
    - Egress 
  egress:
   - to:
     - podSelector:
          matchLabels:
           run: db-pod 

controlplane $ 
```
- Another policy to allow incoming connection to db-pod from app-pod.

```
controlplane $ cat newingress.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      run: db-pod
  policyTypes:
    - Ingress 
  ingress:
   - from:
     - podSelector:
          matchLabels:
           run: app-pod
controlplane $ 
```

- Still there is no communication because default deny policy is blocking the dns traffic. There are 2 solution, one is the allow the dns traffice in default policy or provide the ip address of the pod instead of name.

```
controlplane $ kubectl -n development exec -it app-pod -- curl db-pod
^Ccommand terminated with exit code 130
controlplane $ 
```

- However, we are able to connect to the db-pod on the IP address.
```
controlplane $ kubectl -n development get pods  -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
app-pod   1/1     Running   0          22m   192.168.1.3   node01   <none>           <none>
db-pod    1/1     Running   0          21m   192.168.1.4   node01   <none>           <none>
controlplane $ 

controlplane $ kubectl -n development exec -it app-pod -- curl 192.168.1.4
<title>Welcome to nginx!</title>
controlplane $ 
```
- We have not defined ingress/egress communication policy at the moment hence connection is not going through.
```
controlplane $ kubectl -n development exec -it db-pod -- curl 192.168.1.3
command terminated with exit code 130
controlplane $ 
```

## Scenario 3: Create another namepace and app-pod in deelopment namespace will connect to live-pod in production namespace.

- Create namespace production and launch the pod with clusterIP service:

```
controlplane $ kubectl create namespace production
namespace/production created
controlplane $  
 
controlplane $ kubectl -n production run live-pod --image=nginx
pod/live-pod created
controlplane $ 

controlplane $ kubectl -n production expose pod live-pod --port=80 
service/live-pod exposed
controlplane $ 
```

- Create a default deny policy for production namespace.
```
controlplane $ cat default-deny-production.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-production
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
controlplane $ 
```

- Allow traffic from the production namespace to the app-pod in development namespace.

```
controlplane $ cat allow-egress-nc.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-nc
  namespace: development
spec:
  podSelector:
    matchLabels:
      run: app-pod
  policyTypes:
    - Egress
  egress:
   - to:
        - namespaceSelector:
            matchLabels:
              ns: production 
controlplane $ 
```

- Allow traffice from the development pod to the liv-pod in production namespace.

```
controlplane $ cat allow-ingress-ns.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-ns
  namespace: production
spec:
  podSelector:
    matchLabels:
      run: live-pod
  policyTypes:
    - Ingress
  ingress:
   - from:
        - namespaceSelector:
            matchLabels:
              ns: development
controlplane $
```

        
- Now, we are able to connect to the live-pod ip address from the app-pod.
```    
controlplane $ kubectl -n development exec -it app-pod -- curl 192.168.1.5
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
controlplane $
```
