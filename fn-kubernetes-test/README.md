# running fn in kubernetes (minikube)

## prerequisites

- running minikube
- helm

## install fn

1. start: minikube (tested with minikube version: v0.26.1)
2. enable addon:  minikube addons enable registry-creds
3. start helm: helm init
4. download chart: git clone https://github.com/fnproject/fn-helm && cd fn-helm
5. install dependencies: helm dep build fn
6. fix RBAC issue:  set rbac.enabled to true in fn/values.yaml
7. install fn in kubernetes: helm install --name my-release fn
8. WAIT: some of the Pods are failing, but later they will be restarted and work
9. retrieve the name of the Pod running the Fn API: 
export POD_NAME=$(kubectl get pods --namespace default -l "app=my-release-fn,role=fn-service" -o jsonpath="{.items[0].metadata.name}")
10. set up kubectl port-forwarding; this ensures that any local requests to port 8080 are forwarded by kubectl to the pod specified in this command, on port 80:
kubectl port-forward --namespace default $POD_NAME 8080:80 &
11. switch Docker environment to minikube: eval $(minikube docker-env)
12. login: docker login

### run the function
1. enable port forwarding if not already done: export POD_NAME=$(kubectl get pods --namespace default -l "app=my-release-fn,role=fn-service" -o jsonpath="{.items[0].metadata.name}")
2. deploy the function: fn deploy --app kubernetestestr
3. be patient, first request showed a timeout error, later it works
4. check: while true; do sleep 1; curl  http://localhost:8080/r/kubernetestest/fn-kubernetes-test; done

