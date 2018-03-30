eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWs3c3QyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4MTY5NDJiZi0xMzE4LTExZTgtYTZlNi0wMDE2M2UwMDBiNTAiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.HNJVfVz1cRMuTsrCmbwGH5W3se0hTOHlEoJJ2S21uqs__-sr8tPZNJdqn6C8Xy9e_nAayEqzoILWJFfNoDC6CYb93YGV6IsnaZbCV_vtM-LjnR9cL_Ksp3jX0Cdp1QzXSInDOGysey32tEKktyfmfEjs5xUIIdqUKSaKY86GB9oNZiX665PyTXOvKhWtA3M0bd6Q5nW1VNDy2ZtgGUmAjJtSM7CwcxrKgILd7EAsa9_Gce95fkhkRR6imYVYr_oOKr14APw4J_OY82oauuyPX0LIs_rMYCIzRYEBa4vz1za6xi8wZ1sTXdo76lXFDpR3Ufutlx6oneX5nub_BCLikA

1.Environment="KUBELET_EXTRA_ARGS=--hostname_override=cn-zhangjiakou.i-8vb3m0p2i90y06avs9l0 --provider-id=cn-zhangjiakou.i-8vb3m0p2i90y06avs9l0 --cloud-provider=external"
2.Environment="KUBELET_Aliyun_ARGS=--hostname_override=cn-zhangjiakou.i-8vbcl3ryz9e3ghcgg3cf --provider-id=cn-zhangjiakou.i-8vbcl3ryz9e3ghcgg3cf --cloud-provider=external"
3.Environment="KUBELET_Aliyun_ARGS=--hostname_override=cn-zhangjiakou.i-8vb9fdmx48wpnlzwxsfr --provider-id=cn-zhangjiakou.i-8vb9fdmx48wpnlzwxsfr --cloud-provider=external"
```bash
$ META_EP=http://100.100.100.200/latest/meta-data
$ echo `curl -s http://100.100.100.200/latest/meta-data/region-id`.`curl -s http://100.100.100.200/latest/meta-data/instance-id`
```  
410115424@qq.com
iZ8vb3m0p2i90y06avs9l0Z
```bash
kubeadm init --kubernetes-version=v1.9.2 --pod-network-cidr=10.244.0.0/16 --node-name=cn-zhangjiakou.i-8vb3m0p2i90y06avs9l0
``` 

```bash
$ openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
$ openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
# Writing RSA key
$ rm dashboard.pass.key
$ openssl req -new -key dashboard.key -out dashboard.csr
$ openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
$ kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system
$ kubectl create -f admin-user.yaml -f heapster-rbac.yaml  -f heapster.yaml -f kubernetes-dashboard.yaml
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
``` 