# Dynatrace Operator - EasyTrade lab

## Dynakube Cloud Native Full Stack
Setup the variables
    
    export DT_TENANT_URL="https://abcd.live.dynatrace.com"
    export DT_API_TOKEN="XXX"
    export CLUSTER=k3s123

Installation K3S

    curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=v1.27 K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--disable=traefik" sh -s -

Create namepspace

    kubectl create namespace dynatrace
    
Dynatrace operator

    kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.14.2/kubernetes.yaml

Waiting Dynatrace operator

    kubectl -n dynatrace wait pod --for=condition=ready --selector=app.kubernetes.io/name=dynatrace-operator,app.kubernetes.io/component=webhook --timeout=300s
 
Generate dynakube.yaml

    wget  -O dynakube.yaml https://raw.githubusercontent.com/dynatrace-ace-services/easytrade_lab/main/dynakube.yaml
    cat dynakube.yaml

Dynakube installation

    envsubst < dynakube.yaml | kubectl apply -f -

Dynakube validation

    kubectl exec deploy/dynatrace-operator -n dynatrace -- dynatrace-operator troubleshoot

Waiting dynatrace pods

    while [[ `kubectl get pods -n dynatrace | grep activegate | grep "0/"` ]];do kubectl get pods -n dynatrace;echo "==> waiting for activegate pod ready";sleep 3; done

## EasyTrade

Create namespace 

    kubectl create namespace easytrade

Easytrade installation

    git clone https://github.com/Dynatrace/easytrade.git
    cd easytrade
    kubectl -n easytrade apply -f ./kubernetes-manifests
    
Waiting EasyTrade

    while [[ `kubectl get pods -n easytrade | grep frontend | grep "0/"` ]];do kubectl get pods -n easytrade;echo "==> waiting for frontend pod ready";sleep 3; done
  
## Setup Dynatrace
Additionnal configurations recommanded:  
 - from the K8S settings view : enable monitor events, anomalie detection 
 - follow the recommandation for easytrade : https://github.com/Dynatrace/easytrade

Known limitations:  
 - host k3s is not reconnized as a technologie = "Kubernetes" (softwaretechnologies("KUBERNETES"))
 - impact : the Dashboards "Kubernetes cluster overview" is impacted on 5 tiles
 - workaround : clone the dashboard and use another filter on these tiles

## New release

Get new version of accountservice

    wget  -O accountservice101.yaml https://raw.githubusercontent.com/dynatrace-ace-services/easy_cloudnative_fullstack_deployment/main/accountservice101.yaml

Verify dynatrace variables 

    cat accountservice101.yaml
        
Deploy accountservice 1.0.1 

    kubectl -n easytrade apply -f accountservice101.yaml

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

Add label monitoring=dynatrace

    kubectl label namespace easytrade monitoring=dynatrace

Delete accounservice101

    kubectl delete -f accountservice101.yaml

Restart k3s server

    sudo systemctl restart k3s
    
Uninstall k3s 

    /usr/local/bin/k3s-uninstall.sh

Full clean EasyTravel Docker with OA and AG  

    sudo /opt/dynatrace/oneagent/agent/uninstall.sh
    sudo /opt/dynatrace/gateway/uninstall.sh
    /home/dynatracelab_easytraveld/start-stop-easytravel.sh stop
    sudo rm /etc/init.d/start-stop-easytravel.sh


    
