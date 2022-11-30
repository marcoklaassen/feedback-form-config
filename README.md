# Feedback Form Project


## Prerequisites

### Operator configuration

Install following OpenShift operators:

* Red Hat Integration - AMQ Streams
* Red Hat OpenShift GitOps
* Red Hat OpenShift Pipelines
* Red Hat OpenShift Serverless

Configure the serverless operator with following resources:

```
kind: KnativeEventing
apiVersion: operator.knative.dev/v1alpha1
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec: {}
---
kind: KnativeServing
apiVersion: operator.knative.dev/v1alpha1
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

Create the `feedback-app` namespace:

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

## Config Repository
