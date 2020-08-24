# How to Setup Private Kubernetes Cluster on GCP

## Steps
1. Create **VPC Network**
   * Give name 
   * Add Subnet range (10.10.0.0/16)
   * Add Secondary IP Ranges (10.11.0.0/16) Pods and services(10.12.0.0/16)
2. Create **Cloud NAT Gateway** 
   * Create **Router** For VPC Network
3. Create Cluster By Refering these VPC Network 
   * enable private cluster
   * add master authorized Network
4. get-credentials of new cluster
   * `gcloud container clusters get-credentials cluster_name --region=region --zone=zone`

5. install cert-manager 
  1. if problems regarding "tiller"
    - ```kubectl create serviceaccount --namespace kube-system tiller```
    - ```kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller```
    - ```kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'```
    
  1.  ```kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.13/deploy/manifests/00-crds.yaml```
  2.  ```helm repo add jetstack/cert-manager```
  3.  ```helm repo update```
  4.  ```helm install --name cert-manager --namespace operations jetstack/cert-manager```
  5.  apply cluster issuer from repo. 
      ``` kubectl apply -f https://github.com/pavankumar-go/k8s-private-cluster-installation-guide/cluster-issuer.yaml```
6. Installing Nginx-Ingress stable release
  1. ```helm repo add nginx-stable https://helm.nginx.com/stable```
  2. ```helm repo update```
  3. ```helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true --set controller.publishService.enabled=true```
7. Installing kubedb
  1. ```edit firewall of master to open port "8443", add IP range of Pods```
  2. ```helm install appscode/kubedb --name kubedb --version 0.12.0 --namespace kubedb```
        OR follow kubedb.com docs `0.12.0`  