[Computer networks](/engineering/computer-networks)
# Kubernetes services - under the hood

## Service resource

Service resource is one of the building blocks of Kubernetes platform, used to expose set of pods via a single hostname (which along with CoreDNS also makes up for service discovery mechanism for k8s).<br>
In most cases we do not care how services are implemented behind the scenes, but as always - some sort of mechanical sympathy plays part here. Services per se are mostly a "declaration of intent", sort of descriptor of what we want k8s to achieve for us without playing with tiny little details. There is no such thing as actual "running services" on k8s.

### Types

There are three types of services used for exposing pods:
- `ClusterIP` used to expose set of pods inside k8s cluster
- `NodePort` used to expose set of pods to outside of k8s cluster via specific port exposed on each of the worker nodes
- `LoadBalancer` used to expose set of pods to outside of k8s cluster via cloud vendor specific load balancer solution (generally via single hostname, whereas for NodePort there is need to know address of every worker node); this type generally uses NodePort type services behind the scenes anyway

There is also an `Ingress` resource, also used to expose pods to the outside world. In most cases it's used together with some sort of load balancer, which deals with layer 4 (TCP), hostname-based load balancing between nodes, whereas ingress deals with layer 7 (HTTP), route-based load balancing to actual pods (via services).

## Usage

Services define selector that will be used to match pods to which traffic will be served in round-robin manner:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp <<<<<
```

They also define port on which it will expose access to pods and target port of pods to which traffic should be routed.<br>
For example this service exposes pods labeled `app: MyApp` on port 80 that will map to pods port 9376:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80 <<<<<
      targetPort: 9376 <<<<<<
```

Calling `http://my-service:80` will result in calling one of the pods behind the service on port 9376.


## Under the hood - kube-proxy

Behind the scenes intent of a `Service` resource is actually implemented using a `kube-proxy`.<br> In the past, default mode for proxy was `userspace` mode, that made kube-proxy work as a "sort of reverse proxy" in front of pods. Nowadays, default (by choice at least) implementation is `iptables`.

On every Kuberenetes node there is a `kube-proxy` deamon running that analyzes defined services (from control plane) and reflects traffic they describe in iptables rules.<br> Interesting thing is that iptables is mostly a firewall and so its expected behaviour is that it reads all of its rules (that are additionally chained) in sequence (as any firewall would do - to not ommit any rule and allow invalid traffic). This can impact larger clusters where more modern solution - `ipvs` - can be used.

So, how does `iptables` implementation actually looks like?<br> To allow having single IP address registered in DNS for all of the pods matching *service selector*, we have a *Virtual IP Address* assigned to Service. This address targets multiple `Endpoints`, which represent actual IP addresses of pods behind the service. Then, for a source port that service exposes there is a iptables rule, that randomly (with `statistic mode random probability 0.xxx%`) load balances traffic to each of the pod, by their ip address from service's endpoints and target port from service descriptor.