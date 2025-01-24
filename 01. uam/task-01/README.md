Base tutorial: https://www.linuxlearninghub.com/creating-a-new-user-in-the-kubernetes-cluster/

Create a Private Key for the User:
mkdir ~/certs

openssl genrsa -out ~/certs/minikube-user.key 4096

nano ~/certs/minikube-user.csr.cnf

[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
CN = minikube-user
O = developers
[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth


openssl req -config ~/certs/minikube-user.csr.cnf -new -key ~/certs/minikube-user.key -nodes -out ~/certs/minikube-user.csr



piVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: minikube-authentication
spec:
  groups:
  - system:authenticated
  request: $(cat ~/certs/minikube-user.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
  signerName: kubernetes.io/kube-apiserver-client




kubectl get csr



kubectl certificate approve minikube-authentication



kubectl get csr minikube-authentication -o jsonpath={.status.certificate} | base64 –decode > ~/certs/minikube-user.crt




apiVersion: v1
clusters:
- cluster:
    certificate-authority: /myfolder/ca.crt
    extensions:
    - extension:
        last-update: Sun, 01 Oct 2023 12:23:21 UTC
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://<ipaddress>:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube-user
  name: minikube-user
current-context: minikube-user
kind: Config
preferences: {}
users:
- name: minikube
  user:
   client-certificate: /myfolder/minikube-user.crt
   client-key: /myfolder/minikube-user.key



kubectl –kubeconfig=kubecnf.yaml cluster-info



apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: minikube-user-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]




apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: minikube-edit-role
  namespace: default
subjects:
- kind: User
  name: minikube-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: minikube-user-role
  apiGroup: rbac.authorization.k8s.io
