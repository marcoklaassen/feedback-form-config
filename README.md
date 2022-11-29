# Feedback Form Project


## Prerequisites

Install following OpenShift operators:

* Red Hat Integration - AMQ Streams
* Red Hat OpenShift GitOps
* Red Hat OpenShift Pipelines
* Red Hat OpenShift Serverless

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
