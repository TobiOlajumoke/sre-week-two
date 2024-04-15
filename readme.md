# Introduction

Mike, your teammate at UpCommerce, has just pushed an update after some changes made to the site's code by the development team. Ever since that push, you have been getting pager alerts that the UpCommerce.com service is down. As SRE team lead, it is your job to handle and resolve problems that happen in the UpCommerce Kubernetes cluster. You will also need to keep your Incident Commander (IC) and Communications Lead (CL) up to date so they can manage UpCommerce's users.

Getting your development environment ready

You will use GitHub Codespaces as your development environment, just like you did for the project in Week 1.

Steps

1. Create a fork of the week's repository. This repo contains the exact code that Mike pushed to UpCommerce's production Kubernetes cluster.

2. When you have created a fork of the week's repository, start a codespace on the main branch. The directory structure of the project is the same as Week 1's.

3. Run the command below in your codespace's terminal to create a single-node, Kubernetes cluster using Minikube: 
```
minikube start
```
4. Once your Minikube cluster is running, enter the command below:
```
kubectl create namespace sre
```

This creates a namespace in your Kubernetes cluster named sre. It is within this namespace that you'll do all the tasks required for this project.

5. Run the commands below to install and activate Prometheus in your sre namespace
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus \
  -f prometheus.yml \
  --namespace sre \
```
6. Run the command below to install and activate Grafana in your sre namespace:
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana \
 --namespace sre \
 --set adminPassword="admin"
```
![alt text](images/steps.png)

7. For this task, you'll need the following services running:

    a. Prometheus server

    b. Prometheus Alertmanager

    c. Prometheus PushGateway

    d. Grafana server

You'll need a split terminal for each of the services above because each of these services are forwarded to a port in your Minikube cluster so that you can view them. Activate the services with the following commands:

a. Prometheus server
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9090
```
b. Prometheus Alertmanager
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9093
```
c. Prometheus PushGateway
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/instance=prometheus,app.kubernetes.io/name=prometheus-pushgateway" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9091
```

d. Grafana server
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 3000
```
8. Run the commands below to create a deployment and service 
```
kubectl apply -f deployment.yml -n sre
kubectl apply -f service.yml -n sre
```
9. Run the commands below to see the status of your UpCommerce deployment:
```
kubectl get deployment -n sre
```
![alt text](<images/SPLIT SCREEN.png>)

By now, you will see that your UpCommerce deployment is failing. This is the main reason why you have been getting firing alerts.


Task

You are required to troubleshoot your deployment and find the reason for which it is failing. When you have found it, write a memo (100–150 words) in a Google Doc to your Incident Commander and Communications Lead, informing them of the cause of the service outage.

You can use this simple template to write up your memo. 

Just click File --> Make a Copy to get started!

[Extension]

Troubleshooting system outages using Grafana

Grafana is an open-source analytics and monitoring solution for time-series data. It allows you to query, visualize, alert on, and understand your metrics, no matter where they are stored. Grafana provides a powerful and flexible platform for creating rich, interactive dashboards that can be used for monitoring and analyzing various systems and applications.

The walkthrough video in the Walkthrough tab on this project page guides you through how to use Grafana in GitHub Codespaces.

# Task

Based on what you have learned in the video above, try to troubleshoot UpCommerce's service using details from your Grafana dashboard.

![alt text](<images/create dashboard.png>)


![alt text](<images/add visualization.png>)

![alt text](<images/create data source.png>)

![alt text](<images/add url.png>)

![alt text](<images/scrool and test.png>)
![alt text](<images/sucessful test.png>)

![alt text](<images/create dasboard with prometheus data source.png>)

![alt text](<images/selcet source.png>)
![alt text](<images/k8s grafna.png>)

using grafana we are able to visualize certain metric that helps us to troubleshoot the real issue
![alt text](<images/grfana 2 dashboard.png>)

now let's run this command in our terminal to check the logs

```
kubectl describe pod upcommerce-app-two -n sre
```

![alt text](images/describe.png)

what we see here is that the scheduler couldn't find a place for the pod because there wasn't enough CPU available on any of the nodes. It also couldn't remove any existing pods to make room for the new one.

Looking at the deployment.yml file 

![alt text](images/cpu.png)

I noticed the CPU is 10 but on my machine github codespace i'm using a 2-core • 8GB RAM • 32GB



