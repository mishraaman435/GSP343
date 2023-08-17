# GSP343
GSP-343: Optimize Costs for Google Kubernetes Engine

Here is the workaround of "Optimize Costs for Google Kubernetes Engine: Challenge Lab". \
run all the command one by one.\
Don't forget to replace the onlineboutique-cluster and optimized-pool with your cluster-name and pool-name given in your lab\
also replace max replica with your. i.e, max=13;\
Task 1: Create our cluster and deploy our app
-
_______________________________________________________________________________________________________________________________________________

ZONE=us-central1-b

gcloud container clusters create onlineboutique-cluster --project=$DEVSHELL_PROJECT_ID --zone=$ZONE --machine-type=e2-standard-2 --num-nodes=2

kubectl create namespace dev

kubectl create namespace prod

git clone https://github.com/GoogleCloudPlatform/microservices-demo.git 

cd microservices-demo && kubectl apply -f ./release/kubernetes-manifests.yaml --namespace dev

kubectl get pods -w --namespace dev

--------------------------------------------------------------------------------
run the same command until all all showing (1/1). ie 21 line command by pressing\
Press Ctrl+C\
when all the command showing (1/1) go for next task.
-
kubectl get svc -w --namespace dev

----------------------------------------------------------
do the same again\ 
Go to Services & Ingress section in Kubernetes Engine\
find frontend-external, wait for sometime\
when status of forntend-external os OK then click on the Endpoint address. It will show\
The page you were on is tying to send you to httt://......\
click on the link\
check the progress of task 1
-

Task 2: Migrate to an Optimized Nodepool
-

gcloud container node-pools create optimized-pool --cluster=onlineboutique-cluster --machine-type=custom-2-3584 --num-nodes=2 --zone=$ZONE

for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do  kubectl cordon "$node"; done

for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node"; done

kubectl get pods -o=wide --namespace=dev

gcloud container node-pools delete default-pool --cluster onlineboutique-cluster --zone $ZONE

Press Y\
And check the progress of task 2
-

Task 3: Apply a Frontend Update
-

kubectl create poddisruptionbudget onlineboutique-frontend-pdb --selector app=frontend --min-available 1 --namespace dev

KUBE_EDITOR="nano" kubectl edit deployment/frontend --namespace dev

Find image under spec\
replace image: gcr.io/qwiklabs-resources/onlineboutique-frontend:v2.1\
and in imagePullPolicy: Always\
Press ctrl+x then Y then enter
-
kubectl get pods -w --namespace dev

check all modules are in Running state else wait for sometime and rerun this command
-

kubectl autoscale deployment frontend --cpu-percent=50 --min=1 --max=13 --namespace dev

kubectl get hpa --namespace dev

gcloud beta container clusters update onlineboutique-cluster --enable-autoscaling --min-nodes 1 --max-nodes 6 --zone $ZONE

Check all your progress
-

{OPTIONAL}
-
kubectl exec $(kubectl get pod --namespace=dev | grep 'loadgenerator' | cut -f1 -d ' ') -it --namespace=dev -- bash -c "export USERS=8000; sh ./loadgen.sh"
