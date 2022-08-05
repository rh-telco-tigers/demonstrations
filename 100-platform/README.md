<h2 id="100100">100.100</h2>
<h3 id="100100usecase">Use Case(s):</h3> 

- Implement pod autoscaling on a given workload with test workload with custom metrics

<h3 id="100100overview">Tests(s):</h3>

- Setting up [MetalLB](https://docs.openshift.com/container-platform/4.10/networking/metallb/about-metallb.html)
- Creating a workload which uses a service type of `LoadBalancer`

---
|  Operator(s)  |  Sub-Components | Details     |
| ------------- | --------------- | ----------- |
| N/A | N/A | Included with OpenShift |
---

<h4 id="100100deploy1"><b>Step 1: Deploy MetalLB</b></h4>


To deploy an example of Horizontal Pod Autoscaler (HPA), we'll include a memory-based sample. If WWT has some other examples or custom metrics they would like to use, please feel free to use to them or contact the Red Hat account team.

<h3 id="100100preparation1">Preparation:</h3>

Let's start with the following deployment, which will be deployed to the `teamblue-dev` project:
```
$ cat << EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: memuser
  name: memuser
  namespace: teamblue-dev
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: memuser
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: memuser
    spec:
      containers:
      - image: quay.io/xphyr/memuser:httplatest
        imagePullPolicy: IfNotPresent
        name: memuser
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "1500Mi"
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
EOF
```

Next, create a corresponding service:
```
$ cat << EOF | oc apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: memuserservice
  name: memuserservice
  namespace: teamblue-dev
spec:
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: memuser
  sessionAffinity: None
  type: ClusterIP
EOF
```

Create a usable route for the service with the following command:
```
oc expose svc/memuserservice --namespace=teamblue-dev
```

Finally, create the Horizontal Pod Autoscaler rule:
```
$ cat << EOF | oc apply -f -
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-metrics-memory
  namespace: teamblue-dev
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: memuser
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
       averageUtilization: 80
       type: Utilization
  behavior:
    scaleDown: 
      policies: 
      - type: Pods 
        value: 4 
        periodSeconds: 10 
      - type: Percent
        value: 50 
        periodSeconds: 10
      selectPolicy: Min 
      stabilizationWindowSeconds: 0 
    scaleUp: 
      policies:
      - type: Pods
        value: 5 
        periodSeconds: 10
      - type: Percent
        value: 12 
        periodSeconds: 10
      selectPolicy: Max
      stabilizationWindowSeconds: 0
EOF
```

<h3 id="100100testing1">Testing:</h3>

The first thing that you will need is the route for the service associated with the deployment. Use the following command to gather this information:
```
❯ oc get route -n teamblue-dev
NAME             HOST/PORT                                                PATH   SERVICES         PORT       TERMINATION     WILDCARD
jenkins          jenkins-teamblue-dev.apps.sno-mec.telco.ocp.run                 jenkins          <all>      edge/Redirect   None
memuserservice   memuserservice-teamblue-dev.apps.sno-mec.telco.ocp.run          memuserservice   8080-tcp                   None
my-nginx         my-nginx-teamblue-dev.apps.sno-mec.telco.ocp.run                my-nginx         <all>                      None
nginx            nginx-teamblue-dev.apps.sno-mec.telco.ocp.run                   nginx            80-tcp     edge/Redirect   None
```

Our testing will be against the URL/route: `http://memuserservice-teamblue-dev.apps.sno-mec.telco.ocp.run`

There are two endpoints to consider for this test:
- `consumemem`
- `clearmem`

To begin the test, open another tabbed CLI instance. In that window, first check the pod count and run a `watch` against the `hba`:
```
❯ oc get pods
NAME                       READY   STATUS      RESTARTS   AGE
jenkins-1-9lngg            1/1     Running     0          25h
jenkins-1-deploy           0/1     Completed   0          25h
memuser-8596f6cb55-62f6c   1/1     Running     0          29m
```

Watch the `hba` status with the following command:
```
watch oc get hpa
```

Now, in the first CLI window, run the following `for` loop:
```
for i in {1..30}; do  curl http://memuserservice-teamblue-dev.apps.sno-mec.telco.ocp.run/consumemem; done
```

Wait until the HPA has a chance to respond to the increased memory/load. At first this load will start off very light, like shown in the following example:
```
Every 2.0s: oc get hpa
Sat Jul  9 19:25:03 2022

NAME                          REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   Deployment/memuser   6%/80%    1         10        1          30m
```

Over time, this load will change:
```
Every 2.0s: oc get hpa                                                                                                                                            MacBook-Pro: Sat Jul  9 19:25:58 2022

NAME                          REFERENCE            TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   Deployment/memuser   427%/80%   1         10        1          31m
```

The pods will start to increase:
```
❯ oc get pods
NAME                       READY   STATUS      RESTARTS   AGE
jenkins-1-9lngg            1/1     Running     0          25h
jenkins-1-deploy           0/1     Completed   0          25h
memuser-8596f6cb55-62f6c   1/1     Running     0          33m
memuser-8596f6cb55-75drc   1/1     Running     0          39s
memuser-8596f6cb55-8g6x4   1/1     Running     0          39s
memuser-8596f6cb55-9mnm8   1/1     Running     0          39s
memuser-8596f6cb55-rbtkx   1/1     Running     0          39s
memuser-8596f6cb55-xq8xl   1/1     Running     0          39s
```

And the replica counts will start to increase:
```
Every 2.0s: oc get hpa                                                                                                                                            MacBook-Pro: Sat Jul  9 19:26:55 2022

NAME                          REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   Deployment/memuser   72%/80%   1         10        6          32m
```

To reset the test, and clear memory you can simply use the `clearmem` endpoint:
```
curl http://memuserservice-teamblue-dev.apps.sno-mec.telco.ocp.run/clearmem
```