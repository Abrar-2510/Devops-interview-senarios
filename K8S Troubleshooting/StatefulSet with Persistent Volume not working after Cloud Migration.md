# Kubernetes StatefulSet + Persistent Volume Interview Scenario

---

# 1️⃣ What is a StatefulSet?

A **StatefulSet** is a Kubernetes workload controller used to manage **stateful applications**.

Stateful applications are applications that need to **store and remember data**, even if the pod restarts.

Examples include:

* Databases (MySQL, PostgreSQL, MongoDB)
* Distributed systems (Kafka, Zookeeper)
* Message queues
* Applications that require persistent data

Unlike Deployments, StatefulSets maintain:

* **Stable pod identities**
* **Stable network names**
* **Persistent storage per pod**
* **Ordered deployment and scaling**

This makes StatefulSets ideal for running databases and other stateful systems in Kubernetes.

---

# 2️⃣ Why Do We Use StatefulSet?

We use StatefulSets when an application requires:

### Stable Pod Identity

Pods have predictable names:

```
mysql-0
mysql-1
mysql-2
```

Even if a pod crashes and restarts, it keeps the **same identity**.

---

### Stable Network Identity

Each pod gets a stable DNS name:

```
mysql-0.mysql-service
mysql-1.mysql-service
```

Other services can always reach the same pod.

---

### Persistent Storage Per Pod

Each pod gets its own **PersistentVolumeClaim (PVC)**.

Example:

```
mysql-0 → pvc-mysql-0
mysql-1 → pvc-mysql-1
mysql-2 → pvc-mysql-2
```

If a pod restarts, the same storage is reused.

---

### Ordered Deployment and Scaling

Pods are created **in order**.

Creation order:

```
mysql-0 → mysql-1 → mysql-2
```

Deletion order:

```
mysql-2 → mysql-1 → mysql-0
```

This is important for clustered systems.

---

# 3️⃣ Simple Mental Model

```
StatefulSet
     ↓
Pod Created
     ↓
PVC Created
     ↓
StorageClass
     ↓
CSI Driver
     ↓
Cloud Disk (EBS/EFS)
     ↓
Persistent Volume
     ↓
Pod Starts
```

If the storage provisioning step fails, the pod will remain **Pending**.

---

# 4️⃣ The Scenario

After migrating an application to the cloud (for example AWS EKS), a **StatefulSet** is deployed.

However, the pod does not start.

Running:

```
kubectl describe pod <pod-name>
```

Shows the error:

```
pod has unbound immediate PersistentVolumeClaims
```

---



# 1️⃣1️⃣ Simple Solution Pattern (Cloud vs Local)

When running Kubernetes locally or on simple clusters, a default storage class like **standard** may automatically provision storage.

Example:

```
storageClassName: standard
```

However, in cloud environments the storage must connect to the **cloud provider storage system**.

### On AWS (EKS)

Use cloud storage services:

* **EBS** → block storage for StatefulSets
* **EFS** → shared storage for multiple pods

Example model:

```
Pod
 ↓
PVC
 ↓
StorageClass (ebs-sc)
 ↓
EBS Volume
```

If the cluster needs to communicate with external storage systems, Kubernetes uses **CSI drivers**.

Examples:

* EBS CSI Driver
* EFS CSI Driver

So the rule is:

```
Local cluster → standard storage class
Cloud (AWS) → EBS or EFS
External storage → use CSI driver
```

---

# 5️⃣ What Does This Error Mean?

This means:

* The pod was created
* The pod requested storage through a **PVC**
* Kubernetes could not bind the PVC to a **PersistentVolume**
* The pod stays **Pending**

So the issue is **storage provisioning**, not the container itself.

---

# 6️⃣ Common Causes After Cloud Migration

### Missing CSI Driver

Cloud storage requires **CSI drivers**.

Examples:

* **EBS CSI Driver** for AWS block storage
* **EFS CSI Driver** for shared file storage

Without the driver, Kubernetes cannot create volumes.

---

### Wrong StorageClass

The StatefulSet may reference a StorageClass that:

* does not exist
* has the wrong provisioner
* belonged to the old cluster

---

### No Default StorageClass

If PVC does not specify a class and the cluster has no default, provisioning fails.

---

### Unsupported Access Mode

Example:

EBS supports:

```
ReadWriteOnce
```

But if the PVC asks for:

```
ReadWriteMany
```

Provisioning fails.

---

### Availability Zone Mismatch

EBS volumes exist in one **availability zone**.

If the pod is scheduled in another zone, the volume cannot attach.

---

### IAM Permission Issues

The CSI driver must have permission to create and attach volumes.

---

# 7️⃣ Troubleshooting Steps

Check pods:

```
kubectl get pods
```

Check PVC:

```
kubectl get pvc
kubectl describe pvc <pvc>
```

Check StorageClass:

```
kubectl get storageclass
```

Check CSI drivers:

```
kubectl get csidrivers
```

Check cluster events:

```
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 8️⃣ Solutions

### Install CSI Driver

Example on AWS:

Install **EBS CSI Driver** so Kubernetes can create EBS volumes dynamically.

---

### Create Correct StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
```

---

### Correct Access Mode

```
ReadWriteOnce
```

---

### Use WaitForFirstConsumer

This ensures volumes are created in the same zone as the pod.

---

### Fix IAM Permissions

Ensure the CSI driver can create and attach volumes.

---

# 9️⃣ When Do We Use CSI Drivers?

CSI drivers allow Kubernetes to communicate with external storage systems.

| Driver     | Purpose                        |
| ---------- | ------------------------------ |
| EBS CSI    | Block storage for StatefulSets |
| EFS CSI    | Shared file storage            |
| Azure Disk | Azure storage                  |
| GCE PD     | Google persistent disk         |

---

### EBS Model

```
Pod
 ↓
PVC
 ↓
EBS Volume
```

---


# 🔟 Interview Answer

> StatefulSet is used for stateful applications that require stable identities and persistent storage. Each pod receives its own PVC. After cloud migration, pods may stay pending with the error 'unbound PersistentVolumeClaims' because Kubernetes cannot provision storage. This usually occurs due to a missing CSI driver, incorrect StorageClass, unsupported access modes, or IAM permission issues. Troubleshooting involves checking the pod, PVC, StorageClass, CSI drivers, and cluster events.


StorageClass defines how storage should be created, while the provisioner is the component that actually creates the storage resource.