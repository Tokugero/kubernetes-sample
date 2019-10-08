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

kubectl proxy --port=8080 &
http://localhost:8080/api/v1/namespaces/sample/services/grafana/proxy/?orgId=1
http://localhost:8080/api/v1/namespaces/sample/services/helloworld/proxy/
http://localhost:8080/api/v1/namespaces/sample/services/kibana/proxy/





