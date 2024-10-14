# CKA-Prep---Configure-Pods-and-Containers

A <b>LimitRange</b> in Kubernetes allows administrators to set resource constraints on objects like Pods within a namespace, helping manage CPU, memory, and storage usage. It can enforce minimum/maximum resource limits, set default values, and define ratios between requests and limits. When a LimitRange is applied, Kubernetes checks requests against these rules; if any object exceeds them, creation fails with an error. This helps prevent single resources from consuming all available resources within a namespace, supporting balanced resource distribution.

In Kubernetes, <b>LimitRange</b> is a way to set boundaries on how much CPU, memory, and storage resources a container or pod can request and use within a namespace. It ensures no single container or pod can consume too much, affecting others.
 <b>Request:</b> This is the minimum amount of resources a container needs. Kubernetes uses this value to decide where to place a pod on a node. If a container requests 200 MiB of memory, Kubernetes will only place it on a node with at least 200 MiB free.
 <b>Limit:</b> This is the maximum resource amount a container is allowed to use. If a container tries to use more than its limit, Kubernetes restricts it. For instance, if a container has a memory limit of 500 MiB, it will not be able to consume more than that, even if more memory is available on the node.

Setting these values ensures efficient use of resources and prevents one container from using up too much CPU or memory, which could negatively impact other containers running on the same node.
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
<pre>
 C:\Users\gonca> vi limitrange.yaml
PS C:\Users\gonca> kubectl apply -f .\limitrange.yaml
limitrange/cpu-resource-constraint created
PS C:\Users\gonca> kubectl get limitrange
NAME                      CREATED AT
cpu-resource-constraint   2024-10-14T10:31:15Z
PS C:\Users\gonca> kubectl describe limitrange
Name:       cpu-resource-constraint
Namespace:  default
Type        Resource  Min   Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---  ---------------  -------------  -----------------------
Container   cpu       100m  1    500m             500m           -</pre>
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
 <pre>
  Create a yaml file, and update the file:
PS C:\Users\gonca> kubectl run example-conflict-with-limitrange-cpu --image=registry.k8s.io/pause:2.0 --dry-run=client -o yaml > demo.yaml
PS C:\Users\gonca> vi .\demo.yaml
PS C:\Users\gonca> kubectl apply -f .\demo.yaml
The Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit of 500m
</pre>
 
<summary>3. See if that Pod is scheduled, and see why ?</summary>
We got the error
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
<pre>
This configration means that LimitRange constraints for CPU resources on containers within a namespace:

Min: Minimum CPU request of 100m (0.1 CPU).
Max: Maximum CPU limit of 1 (1 full CPU).
Default Request: If a container doesn’t specify a CPU request, it defaults to 500m.
Default Limit: If a container doesn’t specify a CPU limit, it defaults to 500m.
Max Limit/Request Ratio: No specified maximum ratio, so requests can equal limits.

And from a pod level this means that  a container in a pod has both a CPU limit and a CPU request set to 700m (700 millicores). This means:

Requests: The container will be guaranteed 700 millicores of CPU, ensuring it has the resources it needs.
Limits: The container cannot exceed 700 millicores of CPU. Even if more CPU is available, the container is restricted to this maximum amount.
</pre>
</details>

