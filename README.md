# CKA-Prep---Configure-Pods-and-Containers

A <b>LimitRange</b> in Kubernetes allows administrators to set resource constraints on objects like Pods within a namespace, helping manage CPU, memory, and storage usage. It can enforce minimum/maximum resource limits, set default values, and define ratios between requests and limits. When a LimitRange is applied, Kubernetes checks requests against these rules; if any object exceeds them, creation fails with an error. This helps prevent single resources from consuming all available resources within a namespace, supporting balanced resource distribution.

Resource  <b>Quotase</b> help administrators manage and limit resource usage in Kubernetes namespaces to prevent any one team from overusing resources. A ResourceQuota specifies limits for CPU, memory, storage, and object counts (like pods or services) within a namespace. If resource requests exceed these quotas, creation fails. Quotas work with LimitRanges to enforce minimum and maximum resource limits and ensure fair resource distribution among teams in a shared cluster.

In Kubernetes, <b>PID limiting</b> helps prevent a pod from using too many process IDs, which could impact node stability. You can set PID limits per pod and reserve PIDs for the operating system and Kubernetes daemons to ensure they run smoothly. These limits are configured on the node level via the kubelet, using --pod-max-pids for pod limits and --system-reserved or --kube-reserved for system processes. Additionally, you can set up PID-based evictions to remove pods that exceed PID thresholds, preventing overall system instability.

Kubernetes provides <b>Node Resource Managers</b> to optimize and coordinate CPU, device, and memory resources on nodes, particularly for latency-critical and high-throughput workloads. The Topology Manager is the central manager that aligns these resources based on policies configured through kubelet. Other specific managers include the CPU Manager, Device Manager, and Memory Manager, each with dedicated policies to handle their respective resources. This setup helps ensure pods with specific requirements can operate efficiently on the node.

<b> Lets make the examples on https://kubernetes.io/docs/concepts/policy Page </b>

<details open>
<summary>LimitRange example </summary>
<summary>1. Create a LimitRange with a manifest file.  Default CPU limit is 500m, defaultRequest CPU limit is 500m, max CPU limit is 1 and min CPU limit is 100m.</summary>
<pre>
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
</pre>
<summary>2. Create a Pod that declares a CPU resource request of 700m, but not a limit.</summary>
<pre>apiVersion: v1
kind: Pod
metadata:
  name: example-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
</pre>
<summary>3. See if that Pod is scheduled, and see why ?</summary>
<pre>Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit</pre>
<summary>4. Set both request and limit, and see if that works now.</summary>
<pre>apiVersion: v1
kind: Pod
metadata:
  name: example-no-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
      limits:
        cpu: 700m
</pre>

</details>

