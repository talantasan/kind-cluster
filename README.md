# Kind Cluster with Istio Mesh Setup Guide

This guide provides step-by-step instructions to create a kind (Kubernetes in Docker) cluster and deploy Istio service mesh.

## Prerequisites

Ensure you have the following installed:
- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **kind**: Install with `brew install kind` (macOS) or see [kind documentation](https://kind.sigs.k8s.io/docs/user/quick-start/)
- **kubectl**: Install with `brew install kubectl` (macOS) or see [kubectl installation](https://kubernetes.io/docs/tasks/tools/)
- **Istio**: Download from the included `istio-1.29.1/` directory or [Istio releases](https://github.com/istio/istio/releases)

## Step 1: Create a Kind Cluster

### 1.1 Create a basic kind cluster:
```bash
kind create cluster --name istio-mesh
```

### 1.2 Verify the cluster is running:
```bash
kubectl cluster-info --context kind-istio-mesh
kubectl get nodes
```

### 1.3 (Optional) Create a cluster with custom configuration:
```bash
cat > kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: istio-mesh
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml
```

## Step 2: Prepare Istio

### 2.1 Navigate to the Istio directory:
```bash
cd istio-1.29.1
```

### 2.2 Add istioctl to PATH (temporary):
```bash
export PATH="$PATH:$(pwd)/bin"
```

### 2.3 Verify istioctl:
```bash
istioctl version
```

## Step 3: Install Istio on the Cluster

### 3.1 Install using the default profile:
```bash
istioctl install --set profile=demo -y
```

**Alternative profiles:**
- `minimal`: Core components only
- `default`: Standard production-ready setup
- `demo`: All features for demonstration
- `ambient`: New sidecar-less data plane

### 3.2 Example with custom profile:
```bash
istioctl install --set profile=default -y
```

## Step 4: Enable Sidecar Injection

### 4.1 Label the default namespace for automatic sidecar injection:
```bash
kubectl label namespace default istio-injection=enabled
```

### 4.2 Verify the label:
```bash
kubectl get namespace -L istio-injection
```

## Step 5: Verify Istio Installation

### 5.1 Check Istio pods are running:
```bash
kubectl get pods -n istio-system
```

### 5.2 Check Istio CRDs are installed:
```bash
kubectl get crd | grep istio
```

### 5.3 Run Istio pre-flight checks:
```bash
istioctl analyze
```

## Step 6: Deploy Sample Application (Optional)

### 6.1 Deploy Bookinfo sample application:
```bash
kubectl apply -f istio-1.29.1/samples/bookinfo/platform/kube/bookinfo.yaml
```

### 6.2 Deploy Bookinfo gateway:
```bash
kubectl apply -f istio-1.29.1/samples/bookinfo/networking/bookinfo-gateway.yaml
```

### 6.3 Check deployed pods:
```bash
kubectl get pods
```

### 6.4 Verify services:
```bash
kubectl get services
```

## Step 7: Access the Application (Optional)

### 7.1 Determine the ingress port:
```bash
kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'
```

### 7.2 Get a worker node IP:
```bash
kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}'
```

### 7.3 Access the application:
```bash
INGRESS_HOST=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
curl "http://$INGRESS_HOST:$INGRESS_PORT/productpage"
```

## Cleanup

### Delete the kind cluster:
```bash
kind delete cluster --name istio-mesh
```

## Additional Resources

- [Istio Documentation](https://istio.io/latest/docs/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Istio Samples](istio-1.29.1/samples/README.md)

## Troubleshooting

### Check Istio version:
```bash
istioctl version
```

### View detailed pod status:
```bash
kubectl describe pod -n istio-system <pod-name>
```

### View pod logs:
```bash
kubectl logs -n istio-system <pod-name>
```

### Run Istio diagnostics:
```bash
istioctl analyze -A
```

### Uninstall Istio (if needed):
```bash
istioctl uninstall --purge
```
