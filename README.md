#kubernetes 阿里云ECS实例安装

##aliyun一键安装脚本
```shell
# https://github.com/kubeup/okdc 
curl -s https://raw.githubusercontent.com/kubeup/okdc/master/okdc-centos.sh|sh
```
* 执行完成之后最后会出现执行node节点安装的脚本，然后copy到node节点执行
* 然后再master节点执行kubectl get nodes查看结果
* 如果出现node节点token验证失败，在master执行kubeadm token list 查看token，如果出现多个token
* 执行 kubeadm reset
* 重新在master执行安装流程

##kubernetes-dashboard安装

```yaml
# Copyright 2015 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Configuration to deploy release version of the Dashboard UI compatible with
# Kubernetes 1.6 (RBAC enabled).
#
# Example usage: kubectl create -f <this_file>

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.6.3
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 9090
  selector:
    k8s-app: kubernetes-dashboard 
```
* 在master节点执行 kubectl create -f kubernetes-dashboard.yaml
* 执行kubectl get svc --namespace=kube-system -o wide 查看端口
* http://ip:端口 

##NFS 安装
```shell
# NFS安装参考链接
http://www.cnblogs.com/jkko123/p/6361476.html?utm_source=itdadao&utm_medium=referral
yum install nfs-utils rpcbind
systemctl start rpcbind.service
vi /etc/exports
    /data/tmp 192.168.1.0/24(rw,async,all_squash)
mkdir -p /data/tmp
chown nfsnobody.nfsnobody /data/tmp
exportfs -rv
mkdir -p /data/tmp2
# 挂载测试
mount -t nfs 192.168.1.233:/data/tmp /data/tmp2
df -h
touch /data/tmp/1.txt
ls /data/tmp2
umount /data/tmp2
```
##PV PVC

```shell
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: nfs-pv
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    storageClassName: slow
    nfs:
      path: /data/nfs # nfs目录
      server: 172.26.58.201 # nfs-server-ip
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 2Gi
---
# This pod mounts the nfs volume claim into /usr/share/nginx/html and
# serves a simple web page.
apiVersion: v1
kind: ReplicationController
metadata:
  name: nfs-web
spec:
  replicas: 1
  selector:
    role: web-frontend
  template:
    metadata:
      labels:
        role: web-frontend
    spec:
      containers:
      - name: web
        image: nginx
        ports:
          - name: web
            containerPort: 80
        volumeMounts:
            # name must match the volume name below
            - name: nfs
              mountPath: "/usr/share/nginx/html"
      volumes:
      - name: nfs
        persistentVolumeClaim:
          claimName: nfs-pv-claim
---
kind: Service
apiVersion: v1
metadata:
  name: nfs-web
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
  selector:
    role: web-frontend
```
* kubectl create -f nfs.yaml
* 在kubernetes-dashboard查看结果

