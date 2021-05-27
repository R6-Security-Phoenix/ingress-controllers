# Ambassador ingress module

## Initial env
```
# start a k8s cluster
minikube start --kubernetes-version=v1.21.0 --profile ambassador --vm=true
# on mac it's better to use --driver=hyperkit

# install ambassador
minikube addons enable ambassador --profile ambassador

# tunnel in a separate terminal
minikube tunnel --profile ambassador

# example app which echoes the requests it receives
kubectl create deployment example-app --image=mendhak/http-https-echo:19

# NodePort service
kubectl expose deployment example-app --type=NodePort --port=8080

# ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-app
  annotations:
    kubernetes.io/ingress.class: ambassador
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-app
                port:
                  number: 8080
EOF
```

### Test the ingress for the example app

```
# copy ingress address
kubectl get ing example-app

curl <copied ingress address>
```

## Development cycle:

[envoy lua filter documentation](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/lua_filter.html)

This example is created with an inline script, edit the module.yaml

```
# apply the new module
kubectl apply -f module.yaml

# validate the applied module
kubectl get ambassador-crds 
```

## test the module

```
curl <copied ingress address>
```
