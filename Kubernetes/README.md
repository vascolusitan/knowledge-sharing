# Digging into Kubernetes

<p align="center">
   <img alt="Container vs VM" src="img/cover.png">
</p>

When it comes to application hosting, the name **Kubernetes** (or **K8s**) is often mentioned. But what is this framework? How can it help us? Is it suitable for my specific use case?

Before answering those questions it is important to understand what is **containerization**.

Containerization involves packaging software code along with a compatible operative system (OS) and the required libraries/dependencies into a lightweight executable, called a **container**.

<p align="center">
    <img alt="Container vs VM" src="img/container vs vm.jfif" width="200">
</p>

Just like a **virtual machine** (**VM**), a container runs consistently on any infrastructure, allowing developers to update and deploy their applications faster and more securely. So, why not just use a VM?

Because containers **virtualize the OS** instead of the hardware required to run the software without problems, making them much faster and resource-efficient than VMs.



## What is Kubernetes?
Returning to the main theme, K8s is an open-source framework that uses **containerization** at its core to facilitate the management, deployment and scalability of an application.



## How is Kubernetes Structured (Architecture)?
How does K8s take advantage of containerization?

Through the **design of a distributed system**, composed of two key components: the **control plane** and **worker nodes**, which together constitute a **cluster**.

<p align="center">
   <img alt="Kubernetes Architecture" src="img/Kubernetes High Level Architecture.png">
</p>

### Control Plane:
Also known as the **Master Node**, or **Cluster Master**, it is reponsible for managing the entire cluster state by coordinating tasks and monitoring the worker nodes. This is achieved through the following sub-components:
 - **API-Server**: 
 The main point of interaction for all administrative tasks. It serves as the gateway (connection point) for the K8s control plane, accessible through APIs (scripts or applications) or command-line tools (kubectl or oc);
 - **Etcd**:
 A well-known open-source distributed key-value database, in this context, it stores the configuration, metadata, and state of the cluster;
 - **Scheduler**:
 Continuously monitors the state of the cluster and makes decisions, like applying a Pod to a worker node, based on the node requirements (CPU and memory usage), 
 <abbr title="Applied to nodes to repel Pods without a matching toleration">taint</abbr>
 /
 <abbr title="Applied to Pods to be able to be scheduled to a node with a matching taint">tolerations</abbr>
  or affinity/anti-affinity rules;
 - **Kube-Controller-Manager**:
 Offers a set of controllers (background threads) that constantly watch over the state of the Kubernetes system. If the system does not match the desired state, the controllers take corrective measures;
 - **Cloud-Controller-Manager**:
 Similar to the Kube-Controller-Manager, it uses controllers to manage node status in Kubernetes but focuses on updating the K8s state based on the cloud state to ensure consistency.

### Worker Node:
The master node guides the worker nodes, which are machines (virtual or physical) that run the actual applications, check their health, control their resources, and assign IP addresses. This is done through the following sub-components:
 - **Kubelet**:
 The brain of the worker node, managing it and its corresponding Pods (start, stop, maintain, health check) while also communicating with the control plane;
 - **Kube-Proxy**:
 Controls the network rules for Pod communication, both inside and outside the cluster, while also performing load balancing across Pods for a 
 <abbr title="An abstraction that groups a set of identical Pods that use kube-proxy rules to manage traffic">**Service**</abbr>;
 - **Pod**:
 The smallest deployable unit, which can contain one or more containers that work together and share resources (network, storage, and specifications on how to run) within the Pod. It’s important to note that Pods are 
 <abbr title="Pods do their job and when they are done they get deleted, if something goes wrong they are replaced by a healthy new Pod">ephemeral</abbr>;
 - **Container Runtime**:
 Responsible for pulling the image (container blueprint) from a registry, instantiating it into a container, and managing the container lifecycle. Essentially, it gets the application up and running in a containerized environment when the container starts.

**Notes:** 
- The image registry **is not** part of the Kubernetes cluster, it is an external service that stores container images, they are immutable templates used to start up (instantiate) containers.

<p align="center">
   <img alt="Kubernetes Cluster & Container Registry" src="img/Kubernetes & Container Registry.png">
</p>



## How does Kubernetes manage containerized applications?
In the previous sections we described what K8s is and how it is structured. But how does this architecture **manage and orchestrate** containerized software? 

Through the usage of 
<abbr title="A set of Pods running containerized applications that represent a service and together form a complete application">**workload</abbr>
resources**, which are objects defined at the API level that work together with the Master Node Controllers to ensure the desired state of the application.
They declare how the software should be deployed, managed and scaled. Here are the main types:

<p align="center">
   <img alt="Kubernetes Workload Resources" src="img/Kubernetes Workload Resources.png">
</p>

 - **ReplicaSet**: Primarily used to maintain a stable set of replica Pods for **stateless applications**. If a Pod fails or is terminated, this resource automatically creates a new one, ensuring high availability and reliability (load distribution);
 - **Deployment**: A higher level of abstraction than ReplicaSets, it’s known for features like rolling updates or rollbacks, enhancing reliability. As new ReplicaSet Pods are created, old ones are terminated, gradually replacing every old Pod. This allows application updates without downtime, improving availability;
 - **StatefulSet**: This resource is useful for maintaining a set of Pods for **stateful applications**. These applications require a unique and stable Persistent Storage/Volume (PV) and network identity, enabling ordered deployment.
 Examples include databases, messaging systems, and logging/monitoring systems (when persistence is required);
 - **DaemonSet**: This resource replicates a Pod that hosts a **background-running application (daemon) across all or a subset of worker nodes**, not just a single node.
 Examples include cluster-level storage, logging systems, monitoring systems, and network plugins;
 - **Job**/**CronJob**: Creates one or more Pods responsible for executing a task successfully. If any of the Pods fail, the Job retries running them until it reaches a specified limit. The tasks can be run in parallel, and when using a CronJob, they can be scheduled.
 Examples include batch processing, migrations, backups, testing, continuous integration, and administrative tasks (configuration updates, patches, cleanup).

**Notes:**
- A workload resource uses an API Server object called **Persistent Volume Claim (PVC)** to bind a Pod to a PV. The **StatefulSet is not** the only resource capable of using PVs. For example, a group of stateless application Pods (like ReplicaSet) can use a shared Persistent Volume to store cache or configurations.



## What are Kubernetes best practices?
A K8s cluster is like a high-performance athlete. The best practices are its strict training regimen, ensuring it’s always in top form. Just as an athlete achieves efficiency in movement, quality in performance, and fault tolerance in adversity, a Kubernetes cluster achieves the same through adherence to these best practices.

<p align="center">
    <img alt="Healthy K8s Cluster" src="img/healthy k8s cluster.jfif" width="200">
</p>

Here are some of the K8s best practices:
 - **General Configuration**: Some of the best practices are specifically related to configuration files, including:
    - **Use the Latest API**: Always use the latest stable API version to avoid deprecation issues and maintain a robust and reliable environment;
    - **Use Version Control**: Store configuration files in a version control system to facilitate rollbacks if problems occur with new versions;
    - **Use YAML Format**: Standardize on YAML format for configuration files, as it is more user-friendly and readable compared to JSON;
    - **Group Related Objects**: While decoupling objects into separate files is a good practice, over-decoupling can introduce complexity. Group related objects in the same file to ensure atomic updates;
    - **Avoid Defaults**: Avoid relying on default values to enhance readability and reduce the risk of errors;
    - **Use Annotations**: Use annotations (key-value pairs) attached to API server objects (e.g., Pods, Services, Deployments) to save configuration descriptions, enhancing clarity and documentation efficiency;
 - **Use Workload Resources**: Ensure Pods are associated with workload resources like ReplicaSets to enable rescheduling in case of failure;
 - **Use Services**: Define endpoints and traffic management policies (e.g., load balancing and routing) for a group of Pods using Services. Use Headless Services (Cluster IP = None) when kube-proxy load balancing is not needed:
    - **Use DNS**: Provides stable and readable names, simplifies and automates service discovery, and aids kube-proxy in efficient network traffic management;
    - **Avoid HostPort and HostNetwork**: These options override built-in network rules, potentially causing security and operational complexities;
 - **Use Kubectl**: As the K8s command-line interface tool (CLI), kubectl enables the automation and management of cluster resources from a single point of control. It is particularly useful for executing commands within CI/CD pipelines;
 - **Use Namespaces**: Organize and separate K8s resources efficiently by using namespaces. They help manage different environments or teams and apply resource quotas and access controls;
 - **Use Labels**: Categorize resources at a finer level of abstraction than namespaces. Use selector labels with workload resources, Services, and kubectl commands.
 Examples: tier (backend/frontend), app name;
 - **Use ConfigMaps and Secrets**: Manage configuration data effectively. Use ConfigMaps for non-sensitive data (e.g., environment variables) and Secrets for secure storage of sensitive data (e.g., usernames, passwords);
 - **Use Logging and Monitoring**: Implement centralized logging and monitoring systems to check the application's health, performance, and security. These strategies aggregate logs and metrics for easy analysis;
 - **Use Probes**: Ensure Pods stay reliable and performant by using health checks:
    - **Liveness Probe**: Checks if the container is healthy through periodic checks at user-specified intervals. If it is not, Kubernetes restarts it using an exponential 
    <abbr title="Consists of implementing a retry logic that spaces out attempts with either a fixed or progressively increased delay, preventing an exceeding load on the system">backoff</abbr>
     policy;
    - **Readiness Probe**: Checks if the container is ready to serve traffic. Like Liveness Probes, it runs periodically and uses a backoff policy when it fails;
    - **Startup Probe**: Checks if the container has successfully started through periodic checks at user-specified intervals. If the probe fails, K8s continues to retry until it succeeds or it reaches a user-defined threshold. This strategy is ideal for slow-starting Pods and only enables the other probes after it passes. It does not have a backoff policy;
 - **Keep High Coesion and Low Coupling**: Key principles for designing modular, maintainable, and scalable systems. Here are some examples:
    - Use separate worker nodes or node pools for different types of services, including separate backend, frontend, and infrastructure services to optimize performance and ensure isolation;
    - Each StatefulSet Pod should use its own Persistent Volume Claim and bind to a dedicated Persistent Volume to guarantee data persistence and isolation;
    - Each Pod should typically manage a single container to maintain high cohesion and simplicity;
    - Deploy one complete application per cluster to assure better isolation, security, and resource management.


 **Notes:**
- The described patterns **do not need to be strictly applied**. Some of them may be resource-intensive and lead to high costs that not everyone can manage. It is essential to balance best practices with practical considerations, adapting these patterns to fit your specific needs and resource constraints.

<p align="center">
    <img alt="Happy K8s Cluster" src="img/happy k8s cluster.jfif" width="200">
</p>


## Conslusion
Kubernetes has revolutionized the way we deploy, manage, and scale applications through containerization. It abstracts the complexities of infrastructure management while giving developers full control over their cluster configurations. This robust orchestration platform enables a much better focus on building applications while ensuring their reliability, scalability, and efficiency.



## Frequent Questions

<p align="center">
    <img alt="Frequent Kubernetes Questions" src="img/frequent questions.jfif" width="200">
</p>

### **Q:** Why do companies tend to use OpenShift over Kubernetes?
**A:** OpenShift builds on Kubernetes but offers added features that simplify and enhance the user experience. It provides a user-friendly interface, an integrated image registry, and robust multi-user management. OpenShift also supports plugins for extended functionality, simplifies CI/CD pipeline integration, and allows for advanced multi-cluster management. Additionally, it comes with built-in security features and enterprise-grade support, making it a more comprehensive solution for organizations needing more than just container orchestration.
### **Q:** Why use a Network Plugin (in a DaemonSet) if the Worker Node already has Kube-Proxy to manage node networking?
**A:** Kube-Proxy manages standard networking tasks, while a network plugin hosted in a DaemonSet (like Calico or Flannel) provides additional features such as network policies.
### **Q:** Why use Cluster Storage (in a DaemonSet) instead of ConfigMaps?
**A:** Cluster Storage or Persistent Volumes need to persist across Pod restarts and re-deployments and handle high volumes of data, unlike ConfigMaps.
### **Q:** What is the real difference between Kube-Proxy and Services when both manage Pod networking?
**A:** Services provide a stable endpoint and define how Pods are grouped and accessed, while Kube-Proxy implements the network rules and routes traffic to the Pods based on the Service definitions.
### **Q:** Does Kubernetes have some kind of Graphical User Interface (GUI)?
**A:** Yes, there are several, with the Kubernetes Dashboard being one of the most well-known. This GUI is excellent for cluster management and troubleshooting.
