
# ALLOW traffic from two namespace to an application

## Example
---
Start a web application in the ns `test`

```
kubectl -n test run web --image=nginx --labels="application=web" --expose --port=80
```



Save the following manifest to `allow-access-from-test.yaml`

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-access-from-ns-test
  namespace: test
spec:
  podSelector:
    matchLabels:
      application: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: test
```

Save the following manifest to `allow-access-from-test2.yaml`

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-access-from-ns-test2
  namespace: test
spec:
  podSelector:
    matchLabels:
      application: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: test2
```

A few remarks about this mainfests:

* `namespace: test` deploy this policy to the `test` namespace
* `podSelector` applies the ingress rule to pods with `application: web`
* Ingress rule matching name of namespace is `test` and `test2`

Now apply in to the cluster:

```
$ kubectl apply -f allow-access-from-test.yaml -f allow-access-from-test2.yaml
```

# Try it out

```
$ kubectl -n test2 run alpine-$RANDOM --rm -i -t --image=alpine -- sh

/ # wget -qO- --timeout=2 http://web.test.svc
<!DOCTYPE html>
<html><head>
...
```

Traffic is allowed

# Cleanup

```
kubectl -n test delete pod,service web
kubectl delete networkpolicy -n test  allow-access-from-ns-test allow-access-from-ns-test2
```