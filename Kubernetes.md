# Learn Kubernetes

## Why needed?
* Manage container workloads
* Container orchestration

## Benefits
* Open source
* Can run nearly anywhere
  * on-prem
  * private data centre
  * public cloud
  * embedded in hardware - raspberry pi
* Self-healing
* auto-scaling
* load balancing
* rolling updates + rollbacks while app in use
* define desired state - will deploy and manage containers to meet the desired state
* over monolith
  * no single point of failure

## Successes
* Pokemon Go - ran on GKE (Google K8s Engine)
  *  launched 2016 - experinenced 50X traffic expected
  *  K8 met demand

## Architecture
* Clusters
  * made up of at least 1 master node and at least 1 worker node
  * Minikube - single worker node - good for test/dev
    * Both master and worker nodes are run on same VM - normally
    * Not good for production
    * simplified version of K8 architecture
  * AKS (Azure K8 Service) - managed service
    * Azure takes care of master node - you can't see or manage this directly
    * Not charged for master - as opposed to GCP and AWS ~10p/hr
    * One VM for each worker node - what you pay for

## Master vs worker nodes
* Master
  * Runs control plane
  * similar to Jenkins server - manages workers and what runs on them
  * How many do you need?
    * if setting up own cluster - min 3
    * if using managed servive, no worries!
* Worker
  * where the containers are deployed and run
  * work on data plane - because they run app/db/etc

## Pros vs cons of using K8 over self managing
Pros:
* Hard to run K8s for production - often takes dedicated teams
* need less experts
* security is taken of - for control plane only
* saves time - support available
* allows focus on delivery

Cons:
* You don't have control over any of the points above

## Control plane vs data plane
* Control
  * Point of entry - GUI or CLI
  * Controller-manager
    * looks for changes in node cluster
    * if updates needed calls API server
  * Scheduler
    * allocates resources and places them
  * etcd
    * provides/stores entire state of cluster- key:value
    * good for recovery - backup lots!
  * API server
    * central hub for all interactions with K8s
    * only component that directly interacts with etcd
* Data
  * kubelet
    * connects to API server - receives instructions for worker nodes
  * kube-proxy
    * configures and updates network rules

## K8 objects
* Container
  * NOT an object
  * used within a pod
* Pod
  * single unit for delivering software -smallest unit in K8s
  * group of 1 or more containers
  * share resources around containers within - storage + networking
  * on data plane/worker node
  * specify how to run containers
  * have internal IPs
  * ephemeral - could lose data
* volume
  * persists data beyond termination of pods
* config map
  * key:value databse to store info
* secret
  * store sensitive info
  * base64 encoded - not great
* namespace
  * if you don't define one for deployment will default - "default"
  * groups resources together - logically
* replica set
  * makes pods - usually identical
  * created by deployment - usually

## Security
* not as good as VMs
* use maintained images - but check who's doing the maintaining
  * Canonical maintain ubuntu
  * pros:
    * better security - regularly patched
    * better stability
    * more documentation and support
    * more likely to adhere to industry standards/best practice
    * could be streamlined for performance - e.g. speed, image size
  * cons:
    * relying on others for updates and patches
* use automated vulnerability scanning on repos - built in on Dockerhub
* use own security scanning tools befre you use them - CI/CD
* never run a container with root
* monitor and log container activity

## Autoscaling
* HPA (horizontal pod autoscaler)
  * automatically scales number of pods e.g. in a replica set
  * configured using k8s yaml definitions
* Cluster autoscaler
  * automatically adjusts size of cluster - number of nodes
  * configured at cluster level
* VPA (vertical pod autoscaler)
  * adjust resources available to the containers in pods
* KEDA (k8s event-driven autoscaling)
  * fine-grained control over autoscaling based on events
  * similar to HPA, but more control over triggers