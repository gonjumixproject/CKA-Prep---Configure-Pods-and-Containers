# CKA-Prep---Configure-Pods-and-Containers

A <b>LimitRange</b> in Kubernetes allows administrators to set resource constraints on objects like Pods within a namespace, helping manage CPU, memory, and storage usage. It can enforce minimum/maximum resource limits, set default values, and define ratios between requests and limits. When a LimitRange is applied, Kubernetes checks requests against these rules; if any object exceeds them, creation fails with an error. This helps prevent single resources from consuming all available resources within a namespace, supporting balanced resource distribution.

Resource Quotas help administrators manage and limit resource usage in Kubernetes namespaces to prevent any one team from overusing resources. A ResourceQuota specifies limits for CPU, memory, storage, and object counts (like pods or services) within a namespace. If resource requests exceed these quotas, creation fails. Quotas work with LimitRanges to enforce minimum and maximum resource limits and ensure fair resource distribution among teams in a shared cluster.

