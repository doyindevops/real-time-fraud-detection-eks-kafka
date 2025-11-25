#  End-to-End Real-Time Fraud Detection System

### **Kafka + AWS EKS + Prometheus/Grafana + MLOps Observability**

## ✨ Personal Project Purpose

This project was built independently as a **hands-on demonstration of my DevOps and MLOps engineering skills**. It showcases my ability to **architect, deploy, monitor, and operate real-time, cloud-native systems** using modern production-grade tooling such as **AWS EKS, Kafka, Kubernetes, Helm, Prometheus, Grafana, and Alertmanager**. It reflects my capacity to **design reliable streaming pipelines, implement observability and alerting, and work end-to-end** across infrastructure, services, and monitoring—similar to real **FinTech or ML production environments**.

---

##  Overview

This repository contains a fully deployed **real-time streaming fraud detection pipeline** running on **AWS EKS**.
It uses **Kafka** to stream simulated financial transactions between a **fraud-producer** and **fraud-consumer**, with full **Kubernetes observability**, **Prometheus alerting**, and **Grafana dashboards**.

The purpose of the project is to simulate a **real-world FinTech / MLOps production workload** where monitoring, alerting, reliability, and observability are critical for detecting failures in a real-time pipeline.

---

##  Features

* Real-time Kafka streaming workload deployed on Kubernetes
* Cloud-native infrastructure on **AWS EKS**
* **Prometheus + Grafana + Alertmanager** observability stack
* Custom alert: **FraudConsumerDown**
* Persistent storage via **EBS CSI driver gp3**
* Namespaced workloads for clean separation
* Fully reproducible deployment using Helm + manifests

---

##  Architecture Diagram (ASCII)

```
   ┌──────────────────────────────────────────────────────────┐
   │                      AWS EKS Cluster                     │
   │                       (eu-west-2)                        │
   └──────────────────────────────────────────────────────────┘
                     │                           │
        ┌────────────┘                           └────────────┐

┌──────────────────────────────┐        ┌──────────────────────────────┐
│      fraud-producer          │        │      fraud-consumer          │
│  Python deployment + Config  │ -----> │ Reads + processes Kafka msg   │
└──────────────────────────────┘        └──────────────────────────────┘
                    │
                    ▼
            ┌────────────────┐
            │ Kafka (Topic)  │
            │  fraud-trans   │
            └────────────────┘

                    │ Metrics / Alerts
                    ▼
      ┌────────────────────────────────────────────┐
      │ Prometheus | Grafana | Alertmanager        │
      └────────────────────────────────────────────┘

                Alert example → FraudConsumerDown
```

---

##  Project Structure

```bash
real-time-fraud-detection/
│
├── infrastructure/
│   ├── eks-cluster/
│   │   └── eks-cluster.yaml
│   └── addons/
│       ├── metrics-server/
│       └── kong/
│
├── k8s/
│   ├── namespaces/
│   ├── storage/
│   ├── kafka/
│   └── fraud/
│       ├── producer-deployment.yaml
│       ├── consumer-deployment.yaml
│       ├── topic-job.yaml
│       └── configmap.yaml
│
├── monitoring/
│   └── values.yaml
│
└── Images/   # screenshots used throughout this README
```

---

#  Deployment Guide

---

## **PHASE 1 — Create the EKS Cluster**

```bash
eksctl create cluster -f infrastructure/eks-cluster/eks-cluster.yaml
kubectl get nodes
kubectl get ns
```

###  Cluster successfully created

![Cluster Overview](Images/Cluster-overview.png)


![CloudFormation Stack](Images/CLOUDFORMATION-STACK.png)


![CloudFormation view](Images/CLOUDFORMATION.png)


![EC2 Instances](Images/EC2-INSTANCES.png)


---

## **PHASE 2 — Metrics Server & StorageClass Setup**

```bash
kubectl apply -f k8s/namespaces/namespaces.yaml
kubectl apply -f k8s/storage/gp3-csi-storageclass.yaml
```

###  Metrics & node-data ready

![Metrics Addons](Images/ADDONS1.png)

![Metrics Addons 2](Images/ADDON2.png)


![Cluster-Grafana-Metrics](Images/Kubernetes-Monitoring-Prometheus.png)

---

## **PHASE 3 — Deploy Kafka**

```bash
cd k8s/kafka
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install kafka bitnami/kafka -n kafka -f values.yaml

kubectl get pods -n kafka
kubectl get pvc -n kafka
```

###  Successful Kafka deployment

![Kafka Deployments](Images/KAFKA-DEPLOYMENTS.png)

![Broker Running](Images/KAFKA-BROKER.png)



###  Persistent Kafka storage

![EC2 Volumes](Images/EC2-VOLUMES.png)

### Kafka Topic Creation Job

```bash
kubectl apply -f k8s/fraud/topic-job.yaml
```

![Topic Job](Images/KafkaTopic.png)

---

## **PHASE 4 — Deploy Producer & Consumer Apps**

```bash
kubectl apply -f k8s/fraud/configmap.yaml
kubectl apply -f k8s/fraud/producer-deployment.yaml
kubectl apply -f k8s/fraud/consumer-deployment.yaml

kubectl get pods -n fraud
kubectl logs -l app=fraud-producer -n fraud
```

###  Namespaces

![Namespaces Kafka](Images/Namespaces/kafka.png)

![Namespaces Fraud](Images/Namespaces/fraud.png)

###  Producer & consumer deployments running

![Consumer Deployment](Images/Consumer-Deployment.png)

![Producer Deployment](Images/Producer-Deployment.png)


![Consumer Pods Running](Images/consumer-pods-running.png)

![Producer Pod Running](Images/producer-pod-running.png)


![Fraud Consumer Pods](Images/FRAUD-CONSUMER-PODS.png)

---

## **PHASE 5 — Observability (Monitoring & Alerts)**

```bash
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring -f monitoring/values.yaml
kubectl get pods -n monitoring
```

###  Monitoring running

![Monitoring Pods](Images/MONITORING-PODS.png)


![Services](Images/SERVICES-IN-MONITORING-NS.png)


###  Dashboards

![Grafana Dashboards](Images/PROMETHEUS-GRAFANA.png)

![Kafka Monitoring](Images/HEALTHYKAFKA-KUBE-PROMETHEUS-STACK.png)

### Custom Alert (`FraudConsumerDown`)

Metric used:

```promql
kube_deployment_status_replicas_available{namespace="fraud", deployment="fraud-consumer"}
```

 Alert progression:
![Inactive Alert](Images/FRAUD-CONSUMER-INACTIVE.png)

![Alert firing](Images/FRAUD-CONSUMER-FIRING.png)


![Prometheus Alerts](Images/prometheus-alerts.png)



---

#  Tear Down (Cost Saving)

```bash
eksctl delete cluster --name fraud-detection-cluster --region eu-west-2
```

---


# 📌 Next Enhancements

* ML model deployment with Seldon / KFServing
* GitHub Actions + ArgoCD pipeline
* EFK or Loki logging stack

