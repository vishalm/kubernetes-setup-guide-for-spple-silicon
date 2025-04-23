# Setting Up Kubernetes on Apple Silicon (M1/M2/M3)

This guide provides step-by-step instructions for setting up a fully functional Kubernetes environment on Apple Silicon Macs, complete with the Kubernetes Dashboard for easy cluster management.

## Prerequisites

- Apple Silicon Mac (M1, M2, M3)
- [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/) installed
- Terminal access

## Step 1: Enable Kubernetes in Docker Desktop

1. Open Docker Desktop
2. Go to **Settings** (gear icon) > **Kubernetes**
3. Check **Enable Kubernetes**
4. Click **Apply & Restart**
5. Wait for Kubernetes to start (this may take a few minutes)

## Step 2: Install kubectl (Command Line Tool)

The kubectl command-line tool allows you to interact with your Kubernetes cluster. Docker Desktop should install this automatically, but you can verify by running:

```bash
kubectl version --client
```

If kubectl is not installed, you can install it with:

```bash
# Download the kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"

# Make it executable
chmod +x ./kubectl

# Move to your bin directory
mkdir -p ~/bin
mv ./kubectl ~/bin/kubectl

# Add to PATH in your shell profile
echo 'export PATH=$HOME/bin:$PATH' >> ~/.zshrc  # or ~/.bashrc if using bash
source ~/.zshrc  # or ~/.bashrc
```

## Step 3: Install Helm

Helm is the package manager for Kubernetes, making it easy to install applications.

```bash
# Download Helm for arm64 Mac
curl -fsSL -o helm.tar.gz https://get.helm.sh/helm-v3.17.3-darwin-arm64.tar.gz

# Extract it
tar -zxvf helm.tar.gz

# Create bin directory and move helm there
mkdir -p ~/bin
mv darwin-arm64/helm ~/bin/

# Clean up
rm -rf darwin-arm64 helm.tar.gz

# Add to PATH if not already there
echo 'export PATH=$HOME/bin:$PATH' >> ~/.zshrc  # or ~/.bashrc if using bash
source ~/.zshrc  # or ~/.bashrc
```

Verify Helm installation:

```bash
helm version
```

## Step 4: Install Kubernetes Dashboard

The Kubernetes Dashboard provides a web UI for managing your cluster.

```bash
# Add the Kubernetes Dashboard Helm repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

# Update your Helm repositories
helm repo update

# Install the Dashboard
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --create-namespace \
  --namespace kubernetes-dashboard
```

## Step 5: Create Dashboard Admin User

Create a service account with admin privileges to access the Dashboard:

```bash
# Create namespace and service account
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Create ClusterRoleBinding
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

## Step 6: Generate Authentication Token

Create a token for Dashboard authentication:

```bash
# Generate token
kubectl create token admin-user -n kubernetes-dashboard
```

Save this token securely (e.g., in a file named `admin-token.txt`).

## Step 7: Access the Dashboard

Start the proxy to access the Dashboard:

```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

Now access the Dashboard by opening https://localhost:8443 in your browser.

When prompted for authentication, select **Token** and paste the token you generated in Step 6.

## Useful kubectl Commands

```bash
# View all pods
kubectl get pods --all-namespaces

# View all services
kubectl get services --all-namespaces

# View nodes
kubectl get nodes

# Get detailed information about a pod
kubectl describe pod <pod-name> -n <namespace>

# View logs for a pod
kubectl logs <pod-name> -n <namespace>

# Open a shell in a pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

## Cleanup

To uninstall the Dashboard:

```bash
# Remove the Dashboard
helm uninstall kubernetes-dashboard -n kubernetes-dashboard

# Remove the namespace
kubectl delete namespace kubernetes-dashboard

# Remove admin user bindings
kubectl delete clusterrolebinding admin-user
```

## Troubleshooting

### Dashboard Access Issues
If you cannot access the dashboard, ensure the port-forward command is still running.

### Connection Refused
If you get a "connection refused" error, verify that Kubernetes is running in Docker Desktop.

### Certificate Issues
Browser certificate warnings are normal when accessing localhost via HTTPS. You can safely proceed.

---

**Note**: This setup is for local development only. For production environments, additional security considerations are required.
