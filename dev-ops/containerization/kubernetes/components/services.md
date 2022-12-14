## Services

Used to expose Pods to the network. Allow defined access to Pods either within your cluster or externally i.e. a Service is a set of Pods which can be reached by a fixed DNS name or IP address.

An abstraction which defines a logical set of Pods and policy by which to access them, and this abstraction allows Pods to die and replicate in Kubernetes without impacting your application. Defined using YAML or JSON, and enables a loose coupling between dependent Pods.

Although each Pod has a unique IP address, they are not exposed outside the cluster, and a Service is required to route traffic across the Pods. Can be exposed in different ways by specifying a type.

```console
$ kubectl expose deployment/nginx –port 80
$ kubectl expose deployment/httpenv --port 8888 --type LoadBalancer

$ kubectl get services
$ curl [IP_address_of_service]:8888

$ # for Windows:=
$ kubectl --generator=run-pod/v1 tmp-shell --rm -it --image bredfisher/netshoot -- bash
$ curl httpenv:8888
```

### Communication between Containers

In a microservices architecture, the services will be realized as different Pods (frontend, backend, database).

- Containers in the same Pod can connect with each other using localhost, but using different port number.
- Container in a Pod can communicate with another by directly addressing its IP address (brittle approach as Pods are dispensable and can be restarted).
- Recommended approach between containers in different Pods is through Services, and can connect by using DNS name.

### Types

#### Ingress

Ingresses are closely related objects and are used to set up HTTP routes to Services via a load balancer. Also support HTTPS traffic secured by TLS certificates.

#### ClusterIP

Default service type. Single, internal virtual IP allocated. Only reachable from within cluster (nodes and pods).

#### NodePort

Exposes the Service on the same port of each selected Node in the cluster using NAT. Superset of ClusterIP.

```
<NodeIP>:<NodePort>
```

#### LoadBalancer

Standard way to expose a Service to the internet. Creates an external load balancer in the current cloud (only available when infrastructure/cloud provider gives LB) and assigns a fixed, external IP to the Service. Superset of NodePort i.e. creates NodePort and ClusterIP Services.

#### ExternalName

Maps the Service to an external name i.e. foo.bar.example.com, by returning a CNAME record with its value. No proxy is setup. Not used for pods but for giving pods a DNS name to use outside Kubernetes.

### Port Forwarding

Can access a Service without binding it by using Kubectl's integrated port-forwarding functionality. Works without Services i.e. can directly conenct to a Pod in your deployment.

```console
$ kubectl port-forward deployment/nginx 8080:80
$ kubectl port-forward service/nginx 8080:80
```

## Expose Service to Internet

```console
$ minikube service webapi # use URL provided
```

## Example

Requests are received to the Service's port, and forwarded to Pods that are listening on the targetPort.

```
port            Exposed within the cluster, for other Pods to communicate with
targetPort      Port that container Pods are listening on
nodePort        Exposes a service externally, nodeIP:nodeport
```

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.27.0 (b0ed6a2c9)
  creationTimestamp: null
  labels:
    io.kompose.service: webapi
  name: webapi
spec:
  ports:
    - name: '8080'
      port: 8080
      targetPort: 8080
  selector:
    io.kompose.service: webapi
status:
  loadBalancer: {}
```

for some Services, may need to expose more than one port. Must provide port names so that they are unambiguous.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```
