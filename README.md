# NetworkPolicy
This is the repo for Networkpolicy demo's

## Network policy 
```
kubectl create ns dev1
namespace/dev1 created
kubectl create ns dev2
namespace/dev2 created

kubectl run nginx1 --image=nginx -n dev1
kubectl run nginx2 --image=nginx -n dev2
```
Exec into the pod in dev1 namespace and then curl the ip of pod nginx2 in dev 2 namespace to see the traffic flowing. 
Then create the Network policy 

```
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: dev2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

Now do the test again to see the effect of above create networkpolicy.

Now, create another networkpolicy to allow incoming traffic from dev1 namespace to pods in dev2 namespace on port 80

```
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: kube-demo
  namespace: dev2
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: dev1
    ports:
    - protocol: TCP
      port: 80
EOF
```

Check that this connection is only valid from Dev1 pods on port80 by creating a nginx pod in default namespace and tryingto curl dev2 pod which will fail and also create a pod that listens on port 8080 in dev2 namespace and connection will fail. 

### Ciliumnetworkpolicy demo 
Create deployment, service, networkpolicy and test pod.
```
kubectl create configmap nginx-config --from-file=nginx.conf             
kubectl apply -f network.yaml
kubectl apply -f test.yaml
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
```

Now from the test pod do the curl to the service /publi and it should work

Now, create ingress
`kubecltl apply -f ingress.yaml`
This will fail, in order for this to work, create another ciliumclusterwidenetworkpolicy

`kubectlapply -f np.yaml`
