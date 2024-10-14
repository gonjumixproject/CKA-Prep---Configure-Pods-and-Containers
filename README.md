# CKA-Prep---Configure-Pods-and-Containers

A <b>LimitRange</b> in Kubernetes allows administrators to set resource constraints on objects like Pods within a namespace, helping manage CPU, memory, and storage usage. It can enforce minimum/maximum resource limits, set default values, and define ratios between requests and limits. When a LimitRange is applied, Kubernetes checks requests against these rules; if any object exceeds them, creation fails with an error. This helps prevent single resources from consuming all available resources within a namespace, supporting balanced resource distribution.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

