# Configure Liveness, Readiness and Startup Probes.

Kubernetes provides probes (health checks) to monitor and act on the state of Pods (Containers) and to make sure only healthy Pods serve traffic. With 
help of Probes, we can control when a pod should be deemed started, ready for service, or live to serve traffic.


These 3 kind of probes have 3 different use cases. 

Liveness Probe:-
---
1. Liveness probes let Kubernetes know whether your app (running in a container inside Pod) is healthy.
2. Indicates whether the container is running.
3. If app is healthy, Kubernetes will not interfere with pod functioning. If app is unhealthy, Pod will be marked as unhealthy. If a Pod fails health-checks continuously, the Kubernetes terminates the pod and starts a new one.
4. Liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.
5. Kubelet uses liveness probes to know when to restart a container. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy.

<b>Use Cases:</b>
1. Use Liveness Probe to remove unhealthy pods.
2. If you’d like your container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a restartPolicy of Always or OnFailure.
3. Lse a liveness probe for a container that can fail with no possible recovery.
4. If your container cannot crash by itself when there is an unexpected error occur, then use liveness probes. Using liveness probes can overcome some of the bugs the process might have.


<b>Startup Probe:- </b>
---
1. Startup probes let Kubernetes know whether your app (running in a container inside Pod) has properly started.
2. Indicates whether the application within the container is started.
3. Startup probe has higher priority over the two other probe types. Until the Startup Probe succeeds, all the other Probes are disabled.

<b>Use case:</b> 
1. Use Startup Probe for applications that take longer to start.
2. When your app needs additional time to startup, you can use the Startup Probe to delay the Liveness and Readiness Probe.
3. If your container needs to work on loading large data, configuration files, or migrations during startup, you can use a startup probe.
    
<b>Readiness Probe:-</b>
---

1. Readiness probes let Kubernetes know when your app (running in a container inside Pod) is ready to serve traffic.
2. Indicates whether the container is ready to respond to requests.
3. Kubernetes makes sure the readiness probe passes before allowing a service to send traffic to the pod.

<b>Use case:</b> 
1. Use Readiness Probe to detect partial unavailability.
2. If you’d like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe.
3. If you want your container to be able to take itself down for maintenance, you can specify a readiness probe that checks an endpoint specific to readiness that is different from the liveness probe.



### All the probe have the following parameters:

1. initialDelaySeconds : number of seconds to wait before initiating liveness or readiness probes.
2. periodSeconds: how often to check the probe.
3. timeoutSeconds: number of seconds before marking the probe as timing out (failing the health check).
4. successThreshold : minimum number of consecutive successful checks for the probe to pass.
5. failureThreshold : number of retries before marking the probe as failed. For liveness probes, this will lead to the pod restarting. For readiness probes, this will mark the pod as unready.

### How Liveness Probes Work

There are three types of liveness probes in Kubernetes:

1. <b>HTTP GET Probes:</b> These probes send an HTTP GET request to the specified path and port on the container. If the probe receives an HTTP response with a status code between 200 and 399, it considers the container to be healthy.
2. <b>TCP Socket Probes:</b> These probes attempt to establish a TCP connection to the specified port on the container. If the connection is successful, the container is considered healthy.
3. <b>Exec Probes:</b> These probes execute a command within the container. If the command returns a 0 exit status, the container is considered healthy. Non-zero exit statuses indicate that the container is unhealthy.

Reference:<br>
[1]. https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ <br>
[2]. https://blog.devgenius.io/understanding-kubernetes-probes-5daaff67599a <br>
[3]. https://sudipta-deb.in/2023/03/probes-in-kubernetes-liveness-probe-readiness-probe-startup-probe.html
