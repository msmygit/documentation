# Get Started

To create a LangStream control plane, you will need [kubectl](https://kubernetes.io/docs/reference/kubectl/), [helm cli](https://helm.sh/docs/intro/install/), and a running K8s cluster (a default 4 CPU [minikube](https://minikube.sigs.k8s.io/docs/start/) is fine).

### Installation

1. Start a minikube:

```bash
minikube start
```

2. Deploy Langstream from the Helm repository:

```bash
helm repo add langstream https://langstream.github.io/charts
helm repo update
helm install -n langstream --create-namespace langstream langstream/langstream --values helm/examples/simple.yaml
kubectl wait -n langstream deployment/langstream-control-plane --for condition=available --timeout=300s

```

&#x20; 3\. Open the control-plane and api-gateway ports (in separate terminals):

```bash
# Control plane
kubectl port-forward svc/control-plane 8090:8090
```

```bash
# Api gateway
kubectl port-forward svc/api-gateway 8091:8091
```

### Environment setup

1. Install MinIO for local testing:

```
kubectl apply -f helm/examples/minio-dev.yaml
```

2. Port forward minio in a separate terminal:

```
kubectl port-forward pod/minio 9000 9090 -n minio-dev  
```

3. Install Kafka:

```
kubectl create namespace kafka 
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka 
kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka
```

4. Install the LangStream CLI with brew. Other options are available [here](installation/langstream-cli.md).

```bash
brew tap LangStream/langstream
brew install langstream
```

With LangStream installed and your environment set up, you're ready to build an application.