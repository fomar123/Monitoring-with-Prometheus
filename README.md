# Install Prometheus Stack in Kubernetes
##### Created EKS cluster using eksctl:
•	To see cluster nodes
          
       Kubectl get node
     
•	Deploy online microservices 

        Kubectl apply -f config.microservices.yaml

##### Deployed Microservices Application

•	Add helm repository:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

•	Install Prometheus into its own helm space:

       Kubectl create namespace monitoring 

##### Deploy Prometheius stack using helm:

•	Add helm repository:

     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

•	Install Prometheus into its own helm space:

       Kubectl create namespace monitoring 
 
•	Install helm chart: 

      helm install monitoring prometheus-community/kube-prometheus-stack -n monitorting 

##### Monitored stack deployed:

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/b8bf4e70-1a1f-4338-921c-faf9a4afd275">

##### Checked if Prometheus pod is running:  

      kubectl get all -n monitoring 


##### Checked what’s it inside the Prometheus stateful set , alert manager and the operator and copy description to a file:

<img src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/b0489ede-9950-42d2-9700-e36567596f2c">

<img  src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/c8caa599-c31b-4b40-b522-c515285f5865">

<img src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/d1c22171-253a-49b8-84af-d2fe99bdb8d6">

##### Checked the Secret and copy description  to a file: 

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/0c2b1d05-1064-465d-99b4-4eb6fccd3870">


##### Get Prometheus UI:

•	Port forwarding: 

     kubectl port-forward service/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090 &
     

<img  src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/2ae1fb96-3fc9-4414-bbde-1993dfbdf971"> 

<img  src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/54115849-1a98-4443-8a11-cbf2d6199f7e">



# Introduction to Grafana

##### Accessed Grafana UI

•	Port forward:

     kubectl port-forward service/monitoring-grafana 8080:80 -n monitoring

<img src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/28dd1b00-4c87-4252-915a-d9a378a79a98">

<img src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/3cdc04ee-2a9a-4f99-b2a0-7149412b0b10">

##### Test CPU Spike:

##### Trigger CPU spike to and see it in Grafana:

•	Execute script and create a loop for online shop application. Send 10,000 request to online shop application.

•	Deploy pod in cluster an image that executes curl Command :

    kubectl run curl-test --image=radial/busyboxplus:curl -i --tty –rm

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/175db5a1-87e0-4467-b6a8-a36ae4420709">

##### Create a script which curls the application endpoint. The endpoint is the external loadbalancer service endpoint

      for i in $(seq 1 10000)
    
    do
  
     curl ae4aee0715edc46b988c6ce67121bf57-1459479566.eu-west-3.elb.amazonaws.com > test.txt
   
    done

##### In Grafana you’ll see a spike in CPU:

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/633d5451-6813-4c08-8ec8-d64a361b9bea">

# Created own Alert Rules:

##### Alert-rules.yaml

##### Create an alert-rules.yaml file:
•	Add alert rule when a CPU usage is above 50% 

•	Trigger alert for 2min 

•	Prometheus operator extends the Kubernetes API and we create custom Kubernetes resources and in the background the operator takes our custom Kubernetes resource and tells Prometheus to reload the alert rules

•	Add another alert rule when pod is in a crash looped status and keeps restarting more than 5 times

•	Add lables for Prometheus operator to  pick Prometheus rule

•	Apply alert rule

     Kubectl apply -f alert-rules.yaml

<img width="325" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/3fe0e0e4-7eff-44ac-9494-0df60b6d034f">

•	Check logs of Prometheus pod:

     kubectl logs prometheus-monitoring-kube-prometheus-prometheus-0 -n monitoring -c config-reloader
     
•	Check in Prometheus UI for the two rules that was created 

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/25f9b2b9-d5f4-440c-b3c3-a220e19a2911">

•	Trigger HostHighCpuLoad to see if gets fired :

-	Create pod that has an application inside that simulates CPU load
  
-	Deploy container cpustress  from dockerhub  in pod and it will generate a CPU
Stress above 50%

-	Take docker command and translate it to a kubectl command
  
      kubectl run  cpu-test   --image=containerstack/cpustress -- --cpu 4 --timeout 30s --metrics-brief 

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/bf2a50c0-cfe5-4e30-a970-54cff1eb3398">

-	Alert reached more than 50% and was fired

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/4d1b11b8-f5bd-478f-a0a6-2c0cc948a8b0">

# Introduction to Alertmanager

Get alert manager UI:

        Kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093:9093 &

<img width="367" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/dd6c6574-baed-4505-a717-7483dbd38019">

# Configured Alertmanager with Email Receiver

##### Configured Email notifications:

-	Add email address sender and receiver
  
-	Reference secrets in Kubernetes file thast has password value “authPassword” and create secret file to reference it.
  
-	Create secret file in the same namespace as alert-manager-configuration file
  
-	Execure secret file
  
      Kubectl apply -f email-secret.yaml 
-	Execute alert manager gfile

        Kubectl apply -f alert-manager-configuration.yaml
 	
-	Configuration applied in alertmanager 

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/44cce239-2bef-49ce-ac14-8cd188241a97">

##### Triggered Alerts for Email Receiver:
- Update the cpustress to 60 sec
  
    kubectl run  cpu-test   --image=containerstack/cpustress -- --cpu 4 –timeout 60s  --metrics-brief
  
- HostHighCpuLoad in firing state 

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/84835817-5758-40b0-934f-a5b4dede4e1d">


##### Deployed Redis Exporter

     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     
     helm repo add stable https://charts.helm.sh/stable
     
     helm repo update

     helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml


New target was added on Prometheus UI:

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/8b42c286-4a9c-476f-a5a7-a6f83ed35bfb">

# Alert Rules & Grafana Dashboard for Redis

##### Created Prometheus Rules for Redis

##### Created  RedisDown Alert rule:
  
- Check if redis instance is not available 

##### Created rule for redis that  has too many connections

##### Created Prometheus rule

   Kubectl apply -f redis-rules.yaml

##### Checked Prometheus UI for rules

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/d4e287e0-bddb-4107-af47-d33f7227bfab">

##### Triggered RedisDown Alert:

-	Edit redis-cart deployment

 	    kubectl edit deployment redis-cart
-	Set number of replicas to 0
  
##### Alert for RedisDown firing:

<img width="452" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/177f286d-3fa1-41c6-8fd6-3489f073dd28">

##### Import existing Dashboard in Grafana:

-	Import dashboard ID which is 763


# Monitor own App - Part 1

##### Collect & Expose Metrics with Prometheus Client Library:

##### Deploy app into K8s cluster

##### Built Docker Image:

      Docker build -t fomar123/my-rep:nodeapp . 

<img width="388" alt="image" src="https://github.com/fomar123/Monitoring-with-Prometheus/assets/90075757/388624d1-aeb3-419c-8bf6-bee63c5ce4a4">

##### Created Deployment & Service Configuration

##### Created Secret component - type: docker-registry

-	Create docker login secret in the cluster:

 	      kubectl create secret docker-registry my-registry-key --docker-server=https://index.docker.io/v1/ --docker-username=fomar123  --docker-password=""
 	
##### Deployed App into K8s cluster

                Kubectl apply -f k8s-config.yaml

#####  Check if application is exposing metrics in K8s cluster:

          kubectl port-forward svc/nodeapp 3000:3000


# Monitor Own App - Part 2

##### Scrape Own Application Metrics and Configure Own Grafana Dashboard:

##### Deployed ServiceMonitor component:

-	 Add server configuration file in k8s-config.yaml
	 
-	Executre command :
	
-	    kubectl apply -f k8s-config.yaml
  
-	Check target in Prometheus you’ll see monitoring node-app
  

##### Created Grafana Dashboard
