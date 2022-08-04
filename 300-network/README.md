<h2 id="300100">300.100</h2>
<h3 id="300100usecase">Use Case(s):</h3> 

- Create a service type `LoadBalancer` on a bare metal cluster with minimal effort

<h3 id="300100overview">Tests(s):</h3>

- Setting up [MetalLB](https://docs.openshift.com/container-platform/4.10/networking/metallb/about-metallb.html)
- Creating a workload which uses a service type of `LoadBalancer`

---
|  Operator(s)  |  Sub-Components | Details     |
| ------------- | --------------- | ----------- |
| [MetalLB](https://docs.openshift.com/container-platform/4.10/networking/metallb/about-metallb.html) | N/A | MetalLB is an Optional Operator which is co-maintained by Red Hat |
---

<h4 id="300100deploy1"><b>Step 1: Deploy MetalLB</b></h4>

Why use MetalLB? There [answer](https://docs.openshift.com/container-platform/4.9/networking/metallb/about-metallb.html#nw-metallb-when-metallb_about-metallb-and-metallb-operator) is to apply a service of type `LoadBalancer` for a bare metal deployed OpenShift environment. Next to it's original author (David Anderson, who no longer maintains the project), Red Hat is the [_leading contributor_](https://github.com/metallb/metallb/graphs/contributors) to the MetalLB project, and MetalLB is a maintained operator in OpenShift Marketplace.  

Official instructions for installing MetalLB on OpenShift can be found: [HERE](https://docs.openshift.com/container-platform/4.10/networking/metallb/metallb-operator-install.html)

You can create an `AddressPool` using the example below (take care of naming convention and IP address ranges):

<h4 id="300100note1"><b>NOTE:</b></h4>

You will need to have a router set up correctly for BGP in order to use the configurations below, since BGP was also a request for the RFP.

First create the `AddressPool` object:
```
$ cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: AddressPool
metadata:
  name: pool-example-01
  namespace: metallb-system
spec:
  protocol: bgp
  addresses:
  - 192.168.44.10/24
EOF
```

Next, create the `BGPPeer`:
```
$ cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: BGPPeer
metadata:
  namespace: metallb-system
  name: example-peer
spec:
  peerAddress: 192.168.3.1
  peerASN: 64512
  myASN: 64512
  routerID: 192.168.3.70
EOF
```

<h4 id="300100deploy2"><b>Step 2: Deploy Application</b></h4>

Let's start with something super basic. Let's run a simple webserver in the `teamblue-dev` namespace, and then expose that workload as a `LoadBalancer` IP address from the pool that we created earlier. Issue the following command to run the workload:

```
oc run my-nginx --image=nginx --port=80 --namespace=teamblue-dev
```

```
$ cat << EOF | oc apply -f -
kind: Service
apiVersion: v1
metadata:
  name: my-nginx
  namespace: teamblue-dev
  labels:
    run: my-nginx
spec:
  ipFamilies:
    - IPv4
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  internalTrafficPolicy: Cluster
  type: LoadBalancer
  loadBalancerIP: "192.168.44.60"
  ipFamilyPolicy: SingleStack
  sessionAffinity: None
  selector:
    run: my-nginx
EOF
```

Now, you should be able to open a web browser to the IP address of `192.168.44.60` rather than the `route`, or DNS name. You can also verify that the object was deployed successfully by looking at the services in the `teamblue-dev` namespace, and you can see the difference between the application that we deployed above, vs an application that is _not_ leveraging service type `LoadBalancer` (with the Jenkins example):

```
❯ oc get services -n teamblue-dev
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
jenkins        ClusterIP      172.30.73.225   <none>          80/TCP         22h
jenkins-jnlp   ClusterIP      172.30.79.33    <none>          50000/TCP      22h
my-nginx       LoadBalancer   172.30.237.66   192.168.44.60   80:30308/TCP   38s
```

<h4 id="300100summary"><b>COMMENTS:</b></h4>

There are a couple of things that I would like to call out, which may be obvious from our example above.

First, is that `LoadBalancer` gives users/administrators control over the IP address assignments for applications, however DNS entries will need to be maintained separately by whichever team is managing DNS. When OpenShift [`routes`](https://docs.openshift.com/container-platform/4.10/networking/routes/route-configuration.html#nw-creating-a-route_route-configuration) are used, the wildcard DNS entries configured during (or before) the OpenShift installation cover this named routing automatically.

Another consideration is that bare metal service objects such as `LoadBalancer` leverage `NodePorts` in order to translate services to a given External IP address. You can see this behavior from the example above with the port mapping `80:30308/TCP`.

The good news is that OpenShift provides multiple options to users/administrators, and continually looks for ways to improve the platform release after release. This completely driven by our customers, partners, and through the open source community.

Ask your account SA/SSA for more details and to determine the best solution for your specific use case. 