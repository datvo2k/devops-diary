# BUILD A ZERO-DOWNTIME APPLICATION

To achieve zero downtime, apply technique rolling-update.  
Pod's process includes:
- Container creating
- Running
- Terminating
- Terminated  
In the real life, pod takes time to start up, terminate and update routing. These are the things make application downtime.

## Readiness check
Kubernetes provides readiness to application startup because an application needs to load large data or configuration files during startup or depend on external services after startup.  
Create file `deployment.yml`:
```
readinessProbe:
    httpGet:
        path: /
        port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
    successThreshold: 1
    failureThreshold: 3
```
A readiness probe has the following configuration options:
| PARAMETER | DESCRIPTION |
| ------------- | ------------- |
| initialDelaySeconds | Number of seconds between container start and  probe start to allow for services to initialize.  |
| periodSeconds  | Frequency of readiness test.  |
| timeoutSeconds  | Timeout for probe responses. |
| successThreshold  | The number of consecutive success results needed to switch probe status to “Success”. |
| failureThreshold  | The number of consecutive failed results needed to switch probe status to “Failure”.  |

## Liveness check  
Kubernetes provides liveness to check application still running or broken. If broken Liveness recover by being restarted.  
Types of liveness:
- Exec: The probe runs a command inside the container. 
- HTTP: The probe makes an HTTP GET request against a URL in the container. 
- TCP: The probe tries to connect to a specific TCP port inside the container.
- gRPC: gRPC health-checking probes are supported for applications that use gRPC.
Create file `deployment.yml` and using TCP liveness:
```
livenessProbe:
    tcpSocket:
        port: 8080
    initialDelaySeconds: 15
    periodSeconds: 20
```
## Prestop hook:
To solve the problem of updating routing on K8S that takes time, we will interfere with a lifecycle hook of K8S called preStop. The prestop hook is called synchronous before sending the shutdown signal to the pod. The job of this preStop hook is simply to wait a moment for the load balancer to stop forwarding requests to the pod before shutting down the application.
```
lifecycle:
    preStop:
    exec:
        command: ["/bin/bash", "-c", "sleep 15"]
```

