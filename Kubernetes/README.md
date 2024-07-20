# Digging into Kubernetes

![Cover](/img/cover.png)

Often, when it comes to application hosting, the name **Kubernetes** (or **K8s**) is commonly used. But, in reality, what is this framework? How can it help us? Is it good for my specific use case?

Before answering those questions it is important to know what is **containerization**.

It consists in the packaging of the software code along with a compatible operative system (OS) and the required libraries/dependencies into a lightweight executable, called **container**.

<p align="center">
    <img alt="Container vs VM" src="img/container vs vm.jfif" width="200">
</p>

As a **virtual machine** (**VM**) a container runs consistently on any infrastructure, allowing developers to update and deploy their applications faster and more securely, so why not use a VM?

Because containers **virtualize the OS** instead of the hardware required to run the software without problems, making them way faster and resource-efficient than VMs.



## What is Kubernetes?
Returning to the main theme, K8s is an open-source platform that uses **containerization** at its core to facilitate the management, deployment and scalability of an application.



## How is Kubernetes Structured (Architecture)?
How does K8s take advantage of containerization?

Through the **design of a distributed system**, composed of two key components, the **control plane** and **worker nodes**, that together constitute a **cluster**.

![Kubernetes Architecture](/img/Kubernetes%20High%20Level%20Architecture.png)

### Control Plane:
Also known as **Master Node**, or **Cluster Master**, it is reponsible for managing the whole cluster state, by coordinating tasks and monitoring the worker nodes. This is achieved through the usage of some sub-components:
 - **API-Server**: 
 Is the main point of interaction for all administrative tasks, serves as the gateway (connection point) for the K8s control plane, through API (scripts or applications) or command line tools (kubectl or oc);
 - **Etcd**:
 It is a well known open-source distributed key-value database, in this specific case it stores the configuration, metadata and state of the cluster;
 - **Scheduler**:
 This component continuously monitors the state of the cluster and makes decisions, like applying a Pod to a working node, based on the node requirements (CPU and memory usage), 
 <abbr title="Applied to nodes to repel Pods without a matching toleration">taint</abbr>
 /
 <abbr title="Applied to Pods to be able to be scheduled to a node with a matching taint">tolerations</abbr>
  or affinity/anti-affinity rules;
 - **Kube-Controller-Manager**:
 Offers a set of controllers (background threads) that constantly watch over the state of the Kubernetes system, if it does not match the desired state measures will be taken;
 - **Cloud-Controller-Manager**:
 Like the Kube-Controller-Manager it uses controllers to manage the node status of Kubernetes, but in this case only to update K8s state based on the cloud state to keep consistency.

### Worker Node:
A master node is in charge of guiding the worker nodes that are machines (virtual or physical) that run the actual applications, check their health, control their resources and assign IP addresses, through the following sub-components:
 - **Kubelet**:
 Works as the brain of the worker node, managing it and its corresponding Pods (start, stop, maintain, healthcheck), and also for communicating with the control plane;
 - **Kube-Proxy**:
 Controls the network rules for Pods communication for both the inside and outside of the cluster, while also performing load balancing across Pods for a 
 <abbr title="Abstraction that groups a set of identical Pods that use kube-proxy rules to order the traffic">**Service**</abbr>;
 - **Pod**:
 Corresponds to the smallest deployable unit, but can contain one or more containers, that work together and share resources (network, storage and specification on how to run) from within the Pod, it is important to state that Pods are 
 <abbr title="Pods do their job and when they are done they get deleted, if something goes wrong they are replaced by a healthy new Pod">ephemeral</abbr>;
 - **Container Runtime**:
 It is responsible for pulling the image (container blueprint) from a registry, instantiating it into a container and manage the container lifecycle, in essence, it gets the application up and running in a containerized environment when the container starts.

**Notes:** 
- The image registry **is not** part of the Kubernetes cluster, it is an external service that stores container images, they are an immutable template that is used to start up (instantiate) containers.

![Kubernetes Cluster & Container Registry](/img/Kubernetes%20&%20Container%20Registry.png)




## How does Kubernetes manage containerized applications?
In the previous sections it was described what is K8s, and how it is structured, but how does this architecture **manages and orchestrates** containerized softwares? 

Through the usage of 
<abbr title="set of Pods running containerized applications that represent a service and together form a complete application">**workload</abbr>
resources**, which are objects, defined at the API level, that work together with the Master Node Controllers to ensure the desired state of the application.
They declare how the software should be deployed, managed and scaled. Here are the main types:

![Kubernetes Workload Resources](/img/Kubernetes%20Workload%20Resources.png)

 - **ReplicaSet**: Its primary purpose is to maintain a stable set of replica Pods with **stateless applications**. If a Pod fails or is terminated, this resource automatically creates a new one, offering the system high availability and reliability (load distribution);
 - **Deployment**: Is a resource with a higher level of abstraction then ReplicaSets, its main feature is rolling updates or rolling them back, enhancing reliability. While a new ReplicaSet Pod is created, an old one is terminated, gradually replacing every old Pod, allowing application updates without downtime, improving availability;
 - **StatefulSet**: This workload resource is useful for mantaining a set of Pods of **stateful applications**. They require an **unique** and **stable** Persistent Storage/Volume (**PV**) and network identity, making possible an ordered deployment.
 Examples: databases, messaging systems, logging/monitoring systems (when persistence is required)
 - **DaemonSet**: The strategy of this workload resource is to replicate a pod that hosts a **background running application (daemon) across all or a subnet of worker nodes**, not just a single node. 
 Examples: Cluster level storage, logging systems, monitoring systems and network plugins;
 - **Job**/**CronJob**: Creates one or more Pods responsible for executing a task successfully. If any of the Pods fail, the Job retries to run them until it is reached a specified limit. The tasks can be run in parallel and, when using a CronJob, they can be scheduled.
 Examples: Batch processing, migrations, backups, testing, continuous integration and adminsitrative tasks (configuration update, patches, clean up);

**Notes:**
- A workload resource uses an Api Server object called **Persistent Volume Claim** (**PVC**) to bind a Pod to a PV. The **StatefulSet is not** the only resource which is capable of using PVs. For example, a group of stateless application Pods (like ReplicaSet) can use a **shared** Persistent Volume to store cache or configurations.



## What are Kubernetes best practices?
A K8s cluster is like a high-performance athlete. The best practices are its strict training regimen, ensuring it’s always in top form. Just as an athlete achieves efficiency in movement, quality in performance, and fault tolerance in adversity, a Kubernetes cluster achieves the same through adherence to these best practices.

<p align="center">
    <img alt="Healthy K8s Cluster" src="img/healthy k8s cluster.jfif" width="200">
</p>

Here we have some of the K8s best practices:
 - **General Configuration**: Some of the best practices are specifically related to configuration files, from which we have:
    - **Use Latest API**: Always use the latest stable API version to avoid deprecation issues and maintain a robust and reliable environment;
    - **Use Version Control**: Store configuration files in a version control system to facilitate rollbacks if problems occur with new versions;
    - **Use YAML Format**: Standardize on YAML format for configuration files, as it is more user-friendly and readable compared to JSON;
    - **Group Related Objects**: While decoupling objects into separate files is a good practice, over-decoupling can introduce complexity. Group related objects in the same file to ensure atomic updates;
    - **Avoid Defaults**: Avoid relying on default values to enhance readability and reduce the risk of errors;
    - **Use Annotations**: Use annotations (key-value pairs) attached to API server objects (e.g., Pods, Services, Deployments) to save configuration descriptions, enhancing clarity and documentation efficiency;
 - **Use Workload Resources**: Ensure Pods are associated with workload resources like ReplicaSets to enable rescheduling in case of failure;
 - **Use Services**: Define endpoints and traffic management policies (e.g., load balancing and routing) for a group of Pods using Services. Use Headless Services (Cluster IP = None) when kube-proxy load balancing is not needed:
    - **Use DNS**: Provides stable and readable names, simplifies and automates service discovery, and aids kube-proxy in efficient network traffic management;
    - **Avoid HostPort And HostNetwork**: These options override built-in network rules, potentially causing security and operational complexities;
 - **Use Kubectl**: As the K8s command-line interface tool (CLI), kubectl enables the automation and management of cluster resources from a single point of control. It is particularly useful for executing commands within CI/CD pipelines;
 - **Use Namespaces**: Organize and separate K8s resources efficiently by using namespaces. They help manage different environments or teams and apply resource quotas and access controls;
 - **Use Labels**: Categorize resources at a finer level of abstraction than namespaces. Use selector labels with workload resources, Services, and kubectl commands.
 Examples: tier (backend/frontend), app name;
 - **Use ConfigMaps And Secrets**: Manage configuration data effectively. Use ConfigMaps for non-sensitive data (e.g., environment variables) and Secrets for secure storage of sensitive data (e.g., usernames, passwords);
 - **Use Logging And Monitoring**: Implement centralized logging and monitoring systems to check the application's health, performance, and security. These strategies aggregate logs and metrics for easy analysis;
 - **Use Probes**: Ensure Pods stay reliable and performant by using health checks:
    - **Liveness Probe**: Checks if the container is healthy through running periodic checks at user-specified intervals. If it is not, Kubernetes restarts it using an exponential 
    <abbr title="Consists of implementing a retry logic that spaces out attempts with either a fixed or progressively increased delay, preventing an exceeding load on the system">backoff</abbr>
     policy;
    - **Readiness Probe**: Checks if the container is ready to serve traffic. Just like Liveness Probes, it runs periodically and uses a backoff policy when it fails;
    - **Startup Probe**: Checks if the container has successfully started through periodic checks at user-specified intervals. If the probe fails, K8s continues to retry until it succeeds or it reaches a user-defined threshold. This strategy is ideal for slow-starting Pods and only enables the other probes after it passes. It does not have a backoff policy;
 - **Keep High Coesion And Low Coupling**: Key principles for designing modular, maintainable and scalable systems, here are some examples:
    - Use separate worker nodes or node pools for different types of services Including separate backend, frontend, and infrastructure services to optimize performance and ensure isolation;
    - Each StatefulSet Pod should use its own Persistent Volume Claim and bind to a dedicated Persistent Volume to guarantee data persistence and isolation;
    - Each Pod should typically manage a single container to maintain high cohesion and simplicity;
    - Deploy one complete application per cluster to assure better isolation, security, and resource management;


 **Notes:**
- The described patterns **do not need to be strictly applied**, as some of them may be resource-intensive and lead to high costs that not everyone can manage. It is essential to balance best practices with practical considerations, adapting these patterns to fit your specific needs and resource constraints.

<p align="center">
    <img alt="Happy K8s Cluster" src="img/happy k8s cluster.jfif" width="200">
</p>


## Conslusion
Kubernetes has revolutionized the way we deploy, manage, and scale applications through containerization. It abstracts the complexities of infrastructure management while giving developers full control over their cluster configurations. This robust orchestration platform enables a much better focus on building applications while ensuring their reliability, scalability, and efficiency.



## Frequent Questions

### Why do companies tend to use Openshift intead of Kubernetes?
OpenShift is built on top of Kubernetes but offers additional features and tools, such as enhanced security, developer tools, and management resources.
### Why use a Network Plugin, for example in a DaemonSet, if the Worker Node already has Kube-Proxy to manage node network?
Kube-Proxy manages standard networking tasks, while a network plugin hosted in a DaemonSet (like Calico or Flannel) provides additional features such as network policies.
### Why use a Cluster Storage, in a DaemonSet instead of ConfigMaps?
Cluster Storage or Persistent Volumes need to persist across Pod restarts and re-deployments and handle high volumes of data, unlike ConfigMaps.
### What is the real difference between Kube-Proxy and Services when both manage Pods networking?
Services provide a stable endpoint and define how Pods are grouped and accessed, while Kube-Proxy implements the network rules and routes traffic to the Pods based on the Service definitions.
### Does Kubernetes have some king of Graphical User Interface (GUI)?
Yes, there are several, with the Kubernetes Dashboard being one of the most well-known. This GUI is excellent for cluster management and troubleshooting.















