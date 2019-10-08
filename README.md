# Kubernetes Hello World sample 

This is to show a simple log & timeseries monitored helloworld http server using the following technologies:

* Kubernetes/Docker
* Nginx
* Telegraf
* Influx (Warning, no persistent data configured!)
* Grafana
* FluentD
* Elastic Search (Warning, no persistent data configured!)
* Kibana

## General setup steps
ansible-playbook -i hosts playbooks/sshkeycopy.yaml
ansible-playbook -i hosts playbooks/setupbase.yaml
ansible-playbook -i hosts playbooks/preparek8s.yaml 
disable swap (swapoff -a; vim /etc/fstab)
update spec/calico.yaml ipv4pool if needed
update kubadm-config.yaml with correct kubelet version
copy kubeadm-config.yaml to master node 
k8s-master1# kubeadm init --config=kubeadm-config.yaml --upload-certs
scp /etc/kubernetes/admin.conf ~/.kube/config
k8s-node-xx# kubeadm join k8s-master-1:6443 --token 17d777.htxekj98s4nshhy1 --discovery-token-ca-cert-hash sha256:59e08ab0572a1ab5f3a78b2e15d879af3f958bac06dee04bab6b365953704777

## Finally on to the good bits
kubectl apply -f <all the configs>

## To get access to the exposed services:

```
kubectl proxy --port=8080 &
http://localhost:8080/api/v1/namespaces/sample/services/grafana/proxy/?orgId=1
http://localhost:8080/api/v1/namespaces/sample/services/helloworld/proxy/
http://localhost:8080/api/v1/namespaces/sample/services/kibana/proxy/
```

### Running configuration status

```
(kubernetes-sample) tokugero@ansible:~/repos/kubernetes-sample$ kubectl get nodes,services,configmaps,deployments,pods --all-namespaces
NAME                STATUS   ROLES    AGE    VERSION
node/k8s-master-1   Ready    master   172m   v1.16.1
node/k8s-node-1     Ready    <none>   171m   v1.16.1
node/k8s-node-2     Ready    <none>   170m   v1.16.1

NAMESPACE     NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP                  172m
kube-system   service/kube-dns        ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   172m
sample        service/elasticsearch   ClusterIP   10.98.45.0       <none>        9200/TCP                 119m
sample        service/grafana         ClusterIP   10.101.178.253   <none>        3000/TCP                 58m
sample        service/helloworld      ClusterIP   10.106.222.147   <none>        80/TCP                   60m
sample        service/influx          ClusterIP   10.98.206.147    <none>        8086/TCP                 119m
sample        service/kibana          ClusterIP   10.99.91.49      <none>        5601/TCP                 119m

NAMESPACE     NAME                                           DATA   AGE
kube-public   configmap/cluster-info                         2      172m
kube-system   configmap/calico-config                        4      169m
kube-system   configmap/coredns                              1      172m
kube-system   configmap/extension-apiserver-authentication   6      172m
kube-system   configmap/kube-proxy                           2      172m
kube-system   configmap/kubeadm-config                       2      172m
kube-system   configmap/kubelet-config-1.16                  1      172m
sample        configmap/es-config                            2      119m
sample        configmap/helloworldconf                       1      120m
sample        configmap/helloworldhtml                       1      120m
sample        configmap/telegraf                             1      119m
sample        configmap/telegrafnginx                        1      120m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           169m
kube-system   deployment.apps/coredns                   2/2     2            2           172m
sample        deployment.apps/esnode                    1/1     1            1           119m
sample        deployment.apps/grafana                   1/1     1            1           58m
sample        deployment.apps/helloworld                3/3     3            3           120m
sample        deployment.apps/influx                    1/1     1            1           119m

NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-564b6667d7-fgd52   1/1     Running   1          169m
kube-system   pod/calico-node-bxznc                          1/1     Running   0          169m
kube-system   pod/calico-node-gp4zr                          1/1     Running   1          160m
kube-system   pod/calico-node-mpw5q                          1/1     Running   1          169m
kube-system   pod/coredns-5644d7b6d9-4pr4g                   1/1     Running   1          172m
kube-system   pod/coredns-5644d7b6d9-56fq2                   1/1     Running   1          172m
kube-system   pod/etcd-k8s-master-1                          1/1     Running   3          171m
kube-system   pod/fluentd-76gsk                              1/1     Running   0          61m
kube-system   pod/fluentd-fv4d4                              1/1     Running   11         120m
kube-system   pod/fluentd-nmg6z                              1/1     Running   3          120m
kube-system   pod/kube-apiserver-k8s-master-1                1/1     Running   4          171m
kube-system   pod/kube-controller-manager-k8s-master-1       1/1     Running   7          171m
kube-system   pod/kube-proxy-24kj7                           1/1     Running   0          170m
kube-system   pod/kube-proxy-f6ts4                           1/1     Running   1          171m
kube-system   pod/kube-proxy-rwlvg                           1/1     Running   1          172m
kube-system   pod/kube-scheduler-k8s-master-1                1/1     Running   6          171m
sample        pod/esnode-5c6f9dcd79-jznpr                    2/2     Running   0          14m
sample        pod/grafana-7858c5c666-d2rth                   1/1     Running   0          58m
sample        pod/helloworld-8888cff4d-64c5l                 2/2     Running   0          120m
sample        pod/helloworld-8888cff4d-b8qt6                 2/2     Running   0          120m
sample        pod/helloworld-8888cff4d-f6khv                 2/2     Running   0          120m
sample        pod/influx-568fb87754-bh56z                    1/1     Running   0          119m
sample        pod/telegraf-9s7cv                             1/1     Running   0          107m
sample        pod/telegraf-f4nhc                             1/1     Running   0          107m
sample        pod/telegraf-vj75t                             1/1     Running   1          107m
```
