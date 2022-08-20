# SCALING HORIZONTALLY

The kubernetes HPA (Horizontal Pod Autoscaler) automatically scales the number of pods in a deployment, replication controller, or replica set based on that resource's CPU utilization. 

From the most basic perspective, the Horizontal Pod Autoscaler controller operates on the ratio between desired metric value and current metric value:

`desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]`
Ex: number pod current is 3, and CPU load > 50%, and currentMetricValue at 170% => 3*(170/5) = 10.2 ~ 11. 
```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: http-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demoHPA
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```
If CPU load > 70% pod will scale to 10 and when it gets smaller it will decrease by 1. But we will have problems when we drop down right away. Set case workload CPU CONTINUOUS CHANGE chart scale pod will be a spike.
can be explained as follows:  
- taking the CPU usage > 70% for a brief few seconds => scale up.
- And CPU reduced under 70% => scale down.
- CPU CONTINUOUS load higher 70% => scale up.

To avoid scaling down too soon, we’re using HPA’s scaling behavior feature, which controls how quickly our workloads should scale back down once the traffic load becomes less demanding.
```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: http-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demoHPA
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  behavior:
    scaleDown:
      policies:
      - periodSeconds: 20
        type: Pods
        value: 3
      - periodSeconds: 20
        type: Percent
        value: 10
      selectPolicy: Max
      stabilizationWindowSeconds: 10
```
stabilizationWindowSeconds when metrics indicate that target should be scaled down, this algorithm takes a look into previously calculated desired states and uses highest value from specified interval. For scale down default value is 300, maximum value is 3600 (one hour).







