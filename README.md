# Prerequisites

### Install minikube

```
brew install minikube
```

If `which minikube` fails after installation via brew, you may have to remove the old minikube links and link the newly installed binary:

```
brew unlink minikube
brew link minikube
```


### Start the cluster
```
minikube start -p wefox-challenge-cluster
```

### Selete the profile
```
minikube profile wefox-challenge-cluster
```

If `kubectl` does not work use `minikube kubectl --` alternatively you can use aliases

```
alias kubectl="minikube kubectl --"
```

# Deploying the image

## Imperative way



#### Create the pod
```
kubectl run http-echo --image=docker.io/hashicorp/http-echo:0.2.1 --labels="app=wefox-challenge-pod" -- -text='{"hello": "world"}' -listen=":8080"
```

#### Update the image version to 0.2.3
```
kubectl set image pod/http-echo http-echo=docker.io/hashicorp/http-echo:0.2.3
```

#### Expose the pod
First run `minikube tunnel` in a different terminal

```
kubectl expose pod http-echo --port=8080 --target-port=8080 --name=wefox-challenge-service --type=LoadBalancer
```

#### Test the service
```
curl $(minikube service --url wefox-challenge-service)
```


## Declarative way

#### Prepare the files

**pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wefox-challenge-pod
  labels:
    app: wefox-challenge-pod
spec:
  containers:
  - image: hashicorp/http-echo:0.2.1
    name: demo
    ports:
    - containerPort: 8080
    args:
    - -text="{'hello':'world'}"
    - -listen=:8080
```

**service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wefox-challenge-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30000
  selector:
      app: wefox-challenge-pod
```

#### Create the pod

```
kubectl apply -f pod.yaml
```

#### Update the image version to 0.2.3

```
sed 's/0.2.1/0.2.3/' pod.yaml | kubectl apply -f -
```

#### Expose the pod

First run `minikube tunnel` in a different terminal
```
kubectl apply -f service.yaml
```

#### Test the service
```
curl $(minikube ip):30000
```
