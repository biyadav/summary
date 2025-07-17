docker image pull alpine:latest  
docker image ls  
docker container run hello-world  
docker container ls  
docker container ls -a // list all   
docker container rm 0b5a003756ab  
docker container run -it --name ununtu-container ubuntu:16.04 bash  
docker container rm -f $(docker container ls -aq)// a is for all ; q gives container id  
docker System prune -af //remove unused image stopped container unused resources  
docker container run --name v1 --rm -p 80:80 nginx // rm will automtically remove it when container is stopped  
docker container run --name v1 --rm -p 30080:80 -d nginx  
curl localhost:88  
docker container start id  
docker stop container id  
docker container exec -it 3289d0deff5b51664 bash  
docker volume create  v1  
docker volume ls  
docker volume rm volumaename1  
docker container run --name mapped_Container1 -it --rm -v v1:/app -p30080:80 nginx // intercative without bash  
docker container run --name mapped_Container2 -it --rm -v v1:/app -p30080:80 nginx bash  
//bind mount which is not in docker installation directory /var/lib/docker ....  

docker container run --name bindmount -d -p 34567:80 -v "$(pwd)":/usr/share/nginx/html nginx  

//tempfs volume in host memory can be used for configmap  

docker container run --name tmp --mount="type=tmpfs,destination=/app" -it --rm nginx bash  
// alternative to above if not work  
docker container run --name tmp --tmpfs /app -it --rm nginx bash  

//share volume among containers  
docker container run --name src -v /usr/share/nginx/html -p 34567:80 -d nginx  

//read only volume map  
docker container run -it --name ubuntu -v v1:/app:ro ubuntu bash  



Dockerfile  

FROM ubuntu:16.04  

LABEL maintainer=bipin \  
      place=pune

RUN apt-get update && apt-get install wget vim -y  

docker image build -t nvidea . // build an image with Dockerfile in same dir
docker container run -it --name tmpcustome --rm myimage bash  // rum container from image built  
// default ENTRYPOINT is /bin/sh -c  
//ENTRYPOINT will run when container starts for the first time and there should be only one ENTRYPOINT and CMD  but multiple RUN . RUN executes while builing image  

CMD is the argument to ENTRYPOINT CMD["arg1","arg2"]  

Every command in Dockerfile relates to a layer :  
Step 1/3 : FROM ubuntu:16.04  
16.04: Pulling from library/ubuntu  
16c48d79e9cc: Pull complete  
3c654ad3ed7d: Pull complete  
6276f4f9c29d: Pull complete  
a4bd43ad48ce: Pull complete  
Digest: sha256:6239ca16d54d8edd38788aacde7c6426e5999ffbbd7e97ac3cf04d474341d079  
Status: Downloaded newer image for ubuntu:16.04  
 ---> 657d80a6401d  
Step 2/3 : LABEL maintainer=bipin       place=pune  
 ---> Running in b8be8555d087  
Removing intermediate container b8be8555d087  
 ---> bd6a5657f384
Step 3/3 : RUN apt-get update && apt-get install wget vim -y  
 ---> Running in 31bb465d6c95  
 
 
 we can give additional line in custom image and the custom image will have commands from base image also (union of both )  
 
 //push to docke hub  
 docker image  build -t custom .  
 docker image tag amdocs biyadav01/custom  
 docker login -u  
 docker image push biyadav01/custom  
 
 //container logs  
 docker container logs containeridORname  
 //container stats  
 docker container stats containeidORname  
 
 //create network  
 
docker network create --driver bridge tempnetwork  
docker network ls  
docker container run --name c1 -d --net tempnetwork nginx  
docker container run --name c2 -it  --net tempnetwork ubuntu:16.04 bash //ping to c1  
root@4d579e211bee:/#apt-get update  
root@4d579e211bee:/# apt-get install iputils-ping -y  
root@4d579e211bee:/# ping c1  

                 
############################################KUBERNETES########################################
You can bootstrap a cluster as follows:

 1. Initializes cluster master node:  

 kubeadm init --apiserver-advertise-address $(hostname -i)  


 2. Initialize cluster networking:  

 kubectl apply -n kube-system -f \
    "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"  


 3. (Optional) Create an nginx deployment:  

 kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml  
 
 
 
 You can bootstrap a cluster as follows:  

 1. Initializes cluster master node:  

 kubeadm init --apiserver-advertise-address $(hostname -i)  


 2. Initialize cluster networking:  

 kubectl apply -n kube-system -f \
    "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"  


 3. (Optional) Create an nginx deployment:  

 kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml  
 
 
 clusterIp : internal load balancer  for DB not meant to be exposed to external world  
 nodeport : public access cluster wide  
 ingress: expose multiple services with single endpoint ,works at app level  
 load balancer : create ip  for each service multiple endpoint  
 
 scheduler will pick a node if it hase sufficient resources to create pod  
 deployment : for stateless  apps; counterpart of StaefuleSet  
 
 Daemonset : make sure a copy of pod run on each node within cluster. useful for monitoring of node stats  
 
 StaefuleSet: persistent storage is must per pod to maintain persistency  
 
 
 kubectl get nodes  
 
 kubectl run mypod --image httpd --port 80  
 kubectl get pods  with this pod one deployment is created  
 kubectl get deploy  
 when deployment created three resources are creted  :  deploymentent set replica set with one instance  and pod is created  if pod deleted it will create one for this need to delete the deployment itself  
 kubectl get rs  
 kubectl delete pod mypod-6c9767fc87-d9lbv  
 
 kubectl delete deploy mypod  
 kubectl apply -f pod.yaml  
 kubectl exec -it nginx bash  
 kubectl delete -f pod.yaml // delete everything created using file pod.yaml  
 
 
 kubectl label node node01 az=z1  // label node node01 with az=z1  
 master $ kubectl get nodes --show-labels  
NAME     STATUS   ROLES    AGE     VERSION   LABELS  
master   Ready    master   4m51s   v1.14.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
node01   Ready    <none>   4m37s   v1.14.0   az=z1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux

kubectl apply -f deployment.yaml  
kubectl get pod -o wide  
kubectcl expose deployment canary --type NodePort --port 80  
//to access , ip of node port of service  
//rolling update  canary deployment  

kubectl get pods -w // w for continuous watch with changing status  


kubectcl get nodes -o wide // get ip of node  
kubectl get svc  

//canary deployment 


// secret and configmap
kubectl create secret generic mysql-secret --from-literal="passwd=mysqlPass"  

// volumes
run pv.yaml then pvc.yaml

kubectl run mypod --image httpd --port 80 --dry-run -o yaml >1.yaml  // dry run will not run the command -o yaml will output it to yaml
