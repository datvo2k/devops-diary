# DEPLOYMENT STRATEGY TYPES IN KUBERNETES

In this repo, we are going to talk about the following strategies:  
- **Recreate**: Version A is terminated then version B is replace.  
- **Ramped - Rolling update**: Version B is slowly rolled out and - replacing version B.  
- **Blue-Green**: Version B is released alongside version A, then the traffic is switched to version B.  
- **Canary**: Version B is released to a subset of users, then proceed to a full rollout.  
- **A/B testing**: Version B is released to a subset of users under specific condition.

## RECREATE:
Deployment pattern which simply shuts down all the old pods and replaces them with new ones.  
Create file `deployment.yml` :  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
```

## ROLLING UPDATE:
gradually replaces pod instances with newer versions to ensure there are enough new pods to maintain the availability threshold before terminating old pods. Such a phased replacement ensures there is always a minimum number of available pods that enable a safe rollout of updates without causing any downtime.  
Create file deployment.yml :
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 3
  revisionHistoryLimit: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
        timeoutSeconds: 100
        intervalSeconds: 5
        updatePeriodSeconds: 5
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
```
understand the parameters:  
- maxUnavailable: maximum number of unavailable pods during an update.
- maxSurge: maximum number of pods to be created beyond the desired state during the upgrade. 
- timeoutSeconds: the time (in seconds) that waits for the rolling event to timeout. (default 600s)
- intervalSeconds: specifies the time gap in seconds after an update.(default 1s)
- updatePeriodSeconds: time to wait between individual pods migrations or updates.(default 1s)
- revisionHistoryLimit: how much revision history of this deployment you want to keep.
## CANARY:
Canary release is widely adopted in the process of continuous delivery for a variety of benefits, such as:
- Rollout control: Rollout control can be realized over the new application version by allowing a small subset of users to use the canary release before a complete rollout.
- Easy rollback: Rollback can be quickly implemented in case of any bugs or issues.  
I take `nginx ingress` as an example. After set `nginx.ingress.kubernetes.io/canary: true` for an ingress, use the following annotations for canary rules:  
```
nginx.ingress.kubernetes.io/canary-by-header
nginx.ingress.kubernetes.io/canary-by-header-value
nginx.ingress.kubernetes.io/canary-weight
nginx.ingress.kubernetes.io/canary-by-cookie
```
Reference: [Nginx annotations](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md).







