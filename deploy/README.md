This is a kustomization to deploy PrivateBin to a kubernetes cluster

To test in [kind](https://kind.sigs.k8s.io/) try this:

## Build Docker images

```shell
pushd docker

pushd fpm
docker build -t pb-$(basename $PWD):dev .
popd

pushd nginx
docker build -t pb-$(basename $PWD):dev .
popd

popd
```

## Kind cluster

Create test cluster

```shell
cat > cluster.yaml <<EOD
---
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 8000
        protocol: TCP
      - containerPort: 443
        hostPort: 4443
        protocol: TCP
EOD

kind create cluster --config cluster.yaml --kubeconfig kubeconfig --name privatebin
export KUBECONFIG=$PWD/kubeconfig
```

Load docker images

```shell
docker pull k8s.gcr.io/ingress-nginx/controller:v0.46.0@sha256:52f0058bed0a17ab0fb35628ba97e8d52b5d32299fbc03cc0f6c7b9ff036b61a
docker pull docker.io/jettech/kube-webhook-certgen:v1.5.1
kind load docker-image \
    pb-fpm:dev \
    pb-nginx:dev \
    k8s.gcr.io/ingress-nginx/controller:v0.46.0@sha256:52f0058bed0a17ab0fb35628ba97e8d52b5d32299fbc03cc0f6c7b9ff036b61a \
    docker.io/jettech/kube-webhook-certgen:v1.5.1 \
    --name privatebin
```

Deploy ingress controller and wait for it to be ready

```shell
curl -L https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/kind/deploy.yaml | kubectl apply -f -
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Deploy app

```shell
kustomize build deploy/overlays/kind | kubectl apply -f -
```

## Access app

The kind cluster exposes ports 80 and 443 to localhost host ports 8000 and 4443

so, http://localhost:8000 should display the website
