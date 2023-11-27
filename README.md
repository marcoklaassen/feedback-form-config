# Feedback Form Project
This is a Feedback Form demo project. 
If you want to operate the entire demo in one cluster, ignore all differentiaons around *Kafka OpenShift Cluster* and *Application OpenShift Cluster*.

But if you want to have a real world hybrid- & multi-cloud demo, this project is set up to be deployed in two different independent OpenShift clusters. 

## Prerequisites

### Operator configuration

Install following OpenShift operators:

* Kafka OpenShift Cluster
 * Red Hat Integration - AMQ Streams
 * Red Hat OpenShift GitOps

* Application OpenShift Cluster
 * Red Hat OpenShift GitOps  
 * Red Hat OpenShift Pipelines
 * Red Hat OpenShift Serverless

Configure the serverless operator with following resources in the *Application OpenShift Cluster*:

```
kind: KnativeEventing
apiVersion: operator.knative.dev/v1beta1
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec: {}
---
kind: KnativeServing
apiVersion: operator.knative.dev/v1beta1
metadata:
  name: knative-serving
  namespace: knative-serving
spec: {}
---
kind: KnativeKafka
apiVersion: operator.serverless.openshift.io/v1alpha1
metadata:
  name: knative-kafka
  namespace: knative-eventing
spec:
  broker:
    enabled: false
    defaultConfig:
      numPartitions: 10
      replicationFactor: 3
      bootstrapServers: REPLACE_WITH_COMMA_SEPARATED_KAFKA_BOOTSTRAP_SERVERS
  source:
    enabled: true
  sink:
    enabled: true
  channel:
    enabled: false
    bootstrapServers: REPLACE_WITH_COMMA_SEPARATED_KAFKA_BOOTSTRAP_SERVERS
```

### Namespace configuration

Create the `feedback-app` namespace in both clusters:

```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    argocd.argoproj.io/managed-by: openshift-gitops
  name: feedback-app
spec: {}
status: {}
```

## GitOps Applications

| Application | Content | Target Cluster |
|--------|--------|--------|
|Feedback-Form-API|REST Endpoint and Kafka producer|Application OpenShift Cluster|
|Feedback-Form-CICD|Tekton Pipeline components|Application OpenShift Cluster|
|Feedback-Form-Kafka|AMQ Streams components|Kafka OpenShift Cluster|
|Feedback-Form-Middleware|database components|Application OpenShift Cluster|
|Feedback-Form-UI|not implemented yet|Application OpenShift Cluster|
|Feedback-Persistence-API|Backend to persist incoming feedback from kafka-topic in a database|Application OpenShift Cluster|

## CICD configuration

* Add webhook to your github project `https://github.com/marcoklaassen/feedback-form-api/settings/hooks`. Use the event-listener's route from the *Application OpenShift Cluster*.


## Copy Kafka CA Cert and Kafka User Secrets

Copy from cluster with installed and configured AMQ Streams operator

```
oc get secret -n feedback-app -o yaml dev-feedback-app-messaging-cluster-ca-cert | kubectl neat > dev-feedback-app-messaging-cluster-ca-cert.yaml
oc get secret -n feedback-app -o yaml dev-feedback-user | kubectl neat > dev-feedback-user.yaml
```

and apply to cluster with installed middleware and applications

```
oc apply -f dev-feedback-app-messaging-cluster-ca-cert.yaml
oc apply -f dev-feedback-user.yaml
```
