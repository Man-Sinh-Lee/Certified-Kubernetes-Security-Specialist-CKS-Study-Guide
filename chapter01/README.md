To create an external user (not a service account) with read-only privileges on all cluster resources, you'll need to:

1. Create a Kubernetes `ClusterRole` with read-only permissions.
2. Create a `ClusterRoleBinding` to bind the `ClusterRole` to the external user.
3. Create a certificate for the user and set up the kubeconfig file for the user to use the certificate for authentication.

### Step 1: Create a ClusterRole with Read-Only Permissions

Create a `cluster-role-read-only.yaml` file with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-only
rules:
- apiGroups: [""]
  resources: ["pods", "services", "endpoints", "persistentvolumeclaims", "persistentvolumes", "nodes", "configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["extensions"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
  verbs: ["get", "list", "watch"]
```

Apply the `ClusterRole`:

```bash
kubectl apply -f cluster-role-read-only.yaml
```

### Step 2: Generate Certificates for the External User

Generate a private key for the user:

```bash
openssl genrsa -out man-sinh-lee.key 2048
```

Create a certificate signing request (CSR):

```bash
openssl req -new -key man-sinh-lee.key -out man-sinh-lee.csr -subj "/CN=man-sinh-lee/O=read-only"
```

Generate a certificate for the user:

```bash
sudo openssl x509 -req -in man-sinh-lee.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out man-sinh-lee.crt -days 365
```

### Step 3: Create a kubeconfig File for the External User

Create a kubeconfig file for the user:

```bash
kubectl config set-credentials man-sinh-lee --client-certificate=man-sinh-lee.crt --client-key=man-sinh-lee.key
kubectl config set-context man-sinh-lee-context --cluster=kubernetes --namespace=default --user=man-sinh-lee
kubectl config use-context man-sinh-lee-context
```

### Step 4: Create a ClusterRoleBinding

Create a `cluster-role-binding-read-only.yaml` file with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-only-binding
subjects:
- kind: User
  name: man-sinh-lee
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: read-only
  apiGroup: rbac.authorization.k8s.io
```

Apply the `ClusterRoleBinding`:

```bash
kubectl apply -f cluster-role-binding-read-only.yaml
```

To create a namespace and give the user `man-sinh-lee` full permissions on it, you can follow these steps:

1. Create a new namespace.
2. Create a `Role` with full permissions in the new namespace.
3. Create a `RoleBinding` to bind the `Role` to the user `man-sinh-lee`.

### Step 1: Create a New Namespace

Create a namespace called `man-sinh-lee-namespace`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: man-sinh-lee-namespace
```

Apply the namespace:

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: man-sinh-lee-namespace
EOF
```

### Step 2: Create a Role with Full Permissions

Create a `role-full-permissions.yaml` file with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: man-sinh-lee-namespace
  name: full-permissions
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["extensions"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
```

Apply the `Role`:

```bash
kubectl apply -f role-full-permissions.yaml
```

### Step 3: Create a RoleBinding

Create a `role-binding-full-permissions.yaml` file with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: full-permissions-binding
  namespace: man-sinh-lee-namespace
subjects:
- kind: User
  name: man-sinh-lee
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: full-permissions
  apiGroup: rbac.authorization.k8s.io
```

Apply the `RoleBinding`:

```bash
kubectl apply -f role-binding-full-permissions.yaml
```

By following these steps, you create an external user with read-only permissions on all cluster resources. The user can use the generated kubeconfig file to connect to the Kubernetes cluster.

To generate a kubeconfig file for the user `man-sinh-lee`, you need to create a certificate for the user, then generate the kubeconfig file using that certificate. Here are the steps to accomplish this:

### Step 1: Create a Certificate for the User

1. **Create a private key for the user:**

   ```bash
   openssl genrsa -out man-sinh-lee.key 2048
   ```

2. **Create a Certificate Signing Request (CSR):**

   Create a file named `man-sinh-lee-csr.conf` with the following content:

   ```conf
   [ req ]
   default_bits = 2048
   prompt = no
   default_md = sha256
   req_extensions = req_ext
   distinguished_name = dn

   [ dn ]
   CN = man-sinh-lee
   O = AiDevOps

   [ req_ext ]
   subjectAltName = @alt_names

   [ alt_names ]
   DNS.1 = kubernetes
   ```

   Generate the CSR:

   ```bash
   openssl req -new -key man-sinh-lee.key -out man-sinh-lee.csr -config man-sinh-lee-csr.conf
   ```

3. **Sign the CSR with the Kubernetes CA:**

   Assuming you have access to your Kubernetes CA certificate and key (`ca.crt` and `ca.key`), sign the CSR:

   ```bash
   openssl x509 -req -in man-sinh-lee.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out man-sinh-lee.crt -days 365 -extensions req_ext -extfile man-sinh-lee-csr.conf
   ```

### Step 2: Generate the kubeconfig File

1. **Set the necessary variables:**

   ```bash
   CLUSTER_NAME="kubernetes"
   USER_NAME="man-sinh-lee"
   CONTEXT_NAME="man-sinh-lee-context"
   NAMESPACE="man-sinh-lee-namespace"
   SERVER="https://your-kubernetes-api-server:6443"  # replace with your actual server address
   ```

2. **Create the kubeconfig file:**

   ```bash
   kubectl config set-cluster ${CLUSTER_NAME} --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --server=${SERVER} --kubeconfig=man-sinh-lee.kubeconfig
   kubectl config set-credentials ${USER_NAME} --client-certificate=man-sinh-lee.crt --client-key=man-sinh-lee.key --embed-certs=true --kubeconfig=man-sinh-lee.kubeconfig
   kubectl config set-context ${CONTEXT_NAME} --cluster=${CLUSTER_NAME} --namespace=${NAMESPACE} --user=${USER_NAME} --kubeconfig=man-sinh-lee.kubeconfig
   kubectl config use-context ${CONTEXT_NAME} --kubeconfig=man-sinh-lee.kubeconfig
   ```

### Summary of Files

- **Private Key:** `man-sinh-lee.key`
- **CSR:** `man-sinh-lee.csr`
- **Signed Certificate:** `man-sinh-lee.crt`
- **Kubeconfig File:** `man-sinh-lee.kubeconfig`

The `man-sinh-lee.kubeconfig` file should look something like this:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <base64-encoded-ca-cert>
    server: https://your-kubernetes-api-server:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: man-sinh-lee-namespace
    user: man-sinh-lee
  name: man-sinh-lee-context
current-context: man-sinh-lee-context
kind: Config
preferences: {}
users:
- name: man-sinh-lee
  user:
    client-certificate-data: <base64-encoded-user-cert>
    client-key-data: <base64-encoded-user-key>
```

You can now use this kubeconfig file to authenticate as the user `man-sinh-lee`:

```bash
kubectl --kubeconfig=man-sinh-lee.kubeconfig get pods
```