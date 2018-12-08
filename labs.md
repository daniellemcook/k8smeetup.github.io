# Labs
## install kubernetes

安装控制节点 Master
```bash
# apt install -y docker.io

# less /etc/apt/sources.list.d/kubernetes.list
#   deb http://apt.kubernetes.io/ kubernetes-xenial main
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg|apt-key add -
# apt update
# apt install -y kubeadm=1.13.0-00 kubelet=1.13.0-00 # kubectl=1.13.0-00

# wget https://tinyurl.com/yb4xturm -O rbac-kdd.yaml
# wget https://tinyurl.com/y81vqc9g -O calico.yaml
# less calico.yaml
#   - name: CALICO_IPV4POOL_CIDR
#     value: "192.168.0.0/16"

# kubeadm init --pod-network-cidr 192.168.0.0/16 --apiserver-advertise-address 192.168.20.101
# exit
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ less .kube/config
$ kubectl apply -f /root/rbac-kdd.yaml
$ kubectl apply -f /root/calico.yaml
```
配置 kubectl 
```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
```
添加工作节点
默认 kubeadm 生成的 token 只有24小时有效
回到 Master 控制节点
```bash
# ip a|grep inet
# kubeadm token list

# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt|openssl rsa -pubin -outform der 2>/dev/null|openssl dgst -sha256 -hex|sed 's/^.* //'

# kubeadm join 192.168.20.101:6443 --token 8yoi8f.h8cxfeapgu18wxyv --discovery-token-ca-cert-hash sha256:0b50a82c04db981bd3049e73a4c4828e43b46e44e4ee7683dcff68707d8aba65
```

## 基本操作
```bash
kubectl describe node|grep -i taint
kubectl taint nodes --all node-role.kubernetes.io/master-

kubectl create deploy nginx --image=nginx --dry-run -oyaml
kubectl get events
kubectl get deploy nginx --export -oyaml
kubectl expose deployment/nginx --type=LoadBalancer
kubectl exec nginx-xx --printenv|grep KUBERNETES
kubectl scale deploy nginx --replicas=0
kubectl drain node-x --ignore-daemonsets --delete-local-data
kubectl uncordon node-x
```

## Api

```bash
less ~/.kube/config
export client=$(grep client-cert ~/.kube/config|cut -d " " -f 6)
export key=$(grep client-key-data ~/.kube/config|cut -d " " -f 6)
export auth=$(grep certificate-authority-data ~/.kube/config|cut -d " " -f 6)
echo $client|base64 -d - > ./client.pem
echo $key|base64 -d - > ./client-key.pem
echo $auth|base64 -d - > ./ca.pem
kubectl config view|grep server
  server: https://x.x.x.x:6443
curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem https://x.x.x.x:6443/api/v1/pods
kubectl create deploy nginx --image=nginx -ojson > curldeploy.json
curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem https://x.x.x.x:6443/api/v1/namespaces/default/pods -XPOST -H'Content-Type: application/json' -d@curldeploy.json
strace kubectl get ep|grep 6443
python3 -m json.tool v1/serverresources.json|less

kubectl get secrets --all-namespaces
export token=$(kubectl describe secret default-token-xx|grep ^token|cut -f7 -d ' ')
curl https://x.x.x.x:6443/apis --header "Authorization: Bearer $token" -k
curl https://x.x.x.x:6443/api/v1/namespaces --header "Authorization: Bearer $token" -k
kubectl run -ti busybox --image=busybox --restart=Never
kubectl proxy --api-prefix=/ &
```
## 应用操作
```bash
kubectl set image ds ds-one nginx=nginx:1.8.1-alpine
kubectl rollout history ds ds-one
kubectl rollout history ds ds-one --revision=1
kubectl rollout undo ds ds-one --to-revision=1
kubectl rollout status ds ds-one
```

## service

```bash
kubectl get node --show-labels
kubectl label node node-x system=secondOne
kubectl label node node-x system-
kubectl get po -l app=nginx --all-namespaces
kubectl cluster-info
```
## volume

```bash
kubectl create cm colors --from-literal=text=back --from-file=./primary
```
```yaml
env:
- name: mylike
  valueFrom:
    configMapKeyRef:
      name: colors
      key: text
envFrom:
- configMapRef:
    name: colors
```
```yaml
containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: car-vol
      mountPath: /cars
volumes:
  - name: car-vol
    configMap:
      name: fast-car
```

nfs-server
```bash
apt update && apt install -y nfs-kernel-server
mkdir /opt/sfw
chmod 1777 /opt/sfw
bash -c 'echo software > /opt/sfw/hello.txt'
less /etc/exports
  /opt/sfw/ *(rw,sync,no_root_squash,subtree_check)
exportfs -ra
```
nfs-worker
```bash
apt install -y nfs-common
showmount -e nfs-server
  /opt/sfw *
mount x.x.x.x:/opt/sfw /mnt
ls -l /mnt
dd if=/dev/zero of=/opt/sfw/bigfile bs=1M count=300

```

## Scheduling
```bash
kubectl taint node node-x bubba=value:NoSchedule
kubectl taint node node-x bubba-
```

## loging
```bash
journalctl -u kubelet|less
find /var/log -name "*apiserver*log"
```
## Security
```bash
sytemctl status kubelet
less /var/lib/kubelet/config.yaml
ls /etc/kubernetes/manifests/
kubectl -n kube-system get secrets certificate<TAB> -oyaml
kubectl config view
kubectl config set-credentials -h
cp ~/.kube/config ~/cluster-api-config
kubeadm token -h
kubeadm config print-default
```

## Authentication and Authorization

```bash
kubectl create ns development
kubectl create ns production

kubectl config get-contexts
useradd -s /bin/bash DevDan
passwd DevDan

openssl genrsa -out DevDan.key 2048
openssl req -new -key DevDan.key -out DevDan.csr -subj "/CN=DevDan/O=development"
openssl x509 -req -in DevDan.csr 
  -CA /etc/kubernetes/pki/ca.crt 
  -CAkey /etc/kubernetes/pki/ca.key
  -CAcreateserial -out DevDan.crt -days 30
kubectl config set-credentials DevDan --client-certificate=DevDan.crt --client-key=DevDan.key
kubectl config set-context DevDan-context --cluster=kubernetes --namespace=development --user=DevDan
kubectl --context=DevDan get pods
kubectl config get-contexts
less role-dev.yaml
```

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: development
  name: developer
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["list", "get", "watch", "create", "update", "patch", "delete"]
# You can use ["*"] for all verbs
```

```bash
kubectl create -f role-dev.yaml
```
```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: developer-role-binding
  namespace: development
subjects:
- kind: User
  name: DevDan
  apiGroup: ""
roleRef:
  kind: Role
  name: developer
  apiGroup: ""
```
```bash
kubectl apply -f rolebind.yaml
kubectl --context=DevDan-context get pod
kubectl --context=DevDan-context create deploy nginx --image=nginx
```
