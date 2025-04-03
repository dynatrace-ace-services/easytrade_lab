# Dynatrace Operator - EasyTrade lab

## Clean env
Full clean EasyTravel Docker with OA and AG  

    sudo /opt/dynatrace/oneagent/agent/uninstall.sh
    sudo /opt/dynatrace/gateway/uninstall.sh
    /home/dynatracelab_easytraveld/start-stop-easytravel.sh stop
    sudo rm /etc/init.d/start-stop-easytravel.sh

## Dynakube
Setup the variables
    
    export DT_TENANT_URL=https://abcd.live.dynatrace.com
    export DT_API_TOKEN=XXX
    export CLUSTER=k3s123

Installation K3S

    curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=v1.27 K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--disable=traefik" sh -s -

Create namepspace

    kubectl create namespace dynatrace
    
Dynatrace operator & CSI driver

    kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v1.4.1/kubernetes.yaml
    kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v1.4.1/kubernetes-csi.yaml

Waiting for Dynatrace operator ready

    kubectl -n dynatrace wait pod --for=condition=ready --selector=app.kubernetes.io/name=dynatrace-operator,app.kubernetes.io/component=webhook --timeout=300s

Create secret

    kubectl -n dynatrace create secret generic dynakube --from-literal=apiToken=$DT_API_TOKEN  --from-literal=dataIngestToken=$DT_API_TOKEN
 
Generate dynakube.yaml

    wget  -O dynakube.yaml https://raw.githubusercontent.com/dynatrace-ace-services/easytrade_lab/main/dynakube.yaml
    cat dynakube.yaml

Dynakube installation

    envsubst < dynakube.yaml | kubectl apply -f -

Dynakube validation

    kubectl exec deploy/dynatrace-operator -n dynatrace -- dynatrace-operator troubleshoot

Waiting for Dynatrace pods ready

    while [[ `kubectl get pods -n dynatrace | grep activegate | grep "0/"` ]];do kubectl get pods -n dynatrace;echo "==> waiting for activegate pod ready";sleep 3; done

## EasyTrade

Create namespace 

    kubectl create namespace easytrade

Easytrade installation

    git clone https://github.com/Dynatrace/easytrade.git
    cd easytrade
    kubectl -n easytrade apply -f ./kubernetes-manifests/release
    #optional (generate problem automatically) kubectl -n easytrade apply -f ./kubernetes-manifests/problem-patterns
    
Waiting for EasyTrade frontend pods ready

    while [[ `kubectl get pods -n easytrade | grep frontend | grep "0/"` ]];do kubectl get pods -n easytrade;echo "==> waiting for frontend pod ready";sleep 3; done

Get the ip of reverse proxy (look for EXTERNAL-IP of frontendreverseproxy)

    kubectl -n easytrade get svc

## Setup Dynatrace
Additionnal configurations recommanded:  
 - from the K8S settings view : enable monitor events, anomalie detection 
 - follow the recommandation for easytrade : https://github.com/Dynatrace/easytrade

Known limitations:  
 - host k3s is not reconnized as a technologie = "Kubernetes" (softwaretechnologies("KUBERNETES"))
 - impact : the Dashboards "Kubernetes cluster overview" is impacted on 5 tiles
 - workaround : clone the dashboard and use another filter on these tiles

## Create [Kubernetes] tags from labels 
=> [Grant viewer role to service accounts](https://docs.dynatrace.com/docs/shortlink/kubernetes-tagging#viewer)   
Create the following Role and RoleBinding, which allow the default service account to view the necessary metadata about your namespace easytrade  

    wget  -O dynatrace-oneagent-metadata-viewer.yaml https://raw.githubusercontent.com/dynatrace-ace-services/easytrade_lab/main/dynatrace-oneagent-metadata-viewer.yaml
    kubectl -n easytrade create -f dynatrace-oneagent-metadata-viewer.yaml


## New release

Get new version of accountservice

    wget  -O accountservice.yaml https://raw.githubusercontent.com/dynatrace-ace-services/easytrade_lab/main/accountservice.yaml

Verify dynatrace variables and modify vesrion "1.0.xx" and "LABxx"

    vi accountservice.yaml
        
Apply accountservice

    kubectl -n easytrade apply -f accountservice.yaml

Deploy accountservice 

    kubectl rollout restart -n easytrade deployment accountservice

## Usefull commands
    
Show all namesace

    kubectl get ns
    
Service kubernetes 

    kubectl -n easytrade get svc

Delete All pods (to restart the service)

    kubectl delete --all pods -n easytrade
    
Pod dynatrace

    kubectl get pods -n dynatrace

Pod easytrade

    kubectl get pods -n easytrade

Get all pods system

    kubectl get all -n kube-system
    
Stop loadgen

    kubectl -n easytrade scale --replicas=0 deployment/loadgen

Restart services easytrade

    kubectl delete --all pods -n easytrade

Deployment pod accountservice

    kubectl rollout restart -n easytrade deployment accountservice

Describe dynakube 

    kubectl describe  dynakube -n dynatrace

View yaml of a pod

    kubectl get pods -n easytrade <podname> -oyaml

Add label monitoring=dynatrace

    kubectl label namespace easytrade monitoring=dynatrace

Delete accounservice101

    kubectl delete -f accountservice_version.yaml

Restart k3s server

    sudo systemctl restart k3s
    
Uninstall k3s 

    /usr/local/bin/k3s-uninstall.sh

Full clean EasyTravel Docker with OA and AG  

    sudo /opt/dynatrace/oneagent/agent/uninstall.sh
    sudo /opt/dynatrace/gateway/uninstall.sh
    /home/dynatracelab_easytraveld/start-stop-easytravel.sh stop
    sudo rm /etc/init.d/start-stop-easytravel.sh


    
