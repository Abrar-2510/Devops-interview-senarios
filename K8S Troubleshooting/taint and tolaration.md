# Kubernetes Taints and Tolerations

## 1. What is a Taint?

A **taint** is applied to a node to prevent pods from being scheduled on it unless the pod explicitly allows it.

Think of taints as a **"repel" mechanism**.

```
Node → Taint applied
Pods → Cannot run on this node
```

Only pods with the correct **toleration** can run there.

---

## 2. Why Do We Use Taints?

Taints are used when we want to **restrict certain nodes for specific workloads**.

Examples:

| Use Case         | Reason                              |
| ---------------- | ----------------------------------- |
| GPU nodes        | Only GPU workloads should run there |
| Dedicated nodes  | Run only specific applications      |
| Production nodes | Prevent dev workloads               |
| Maintenance      | Temporarily stop scheduling pods    |

---

## 3. Types of Taint Effects

### NoSchedule

Pods **will not be scheduled** on the node unless they tolerate the taint.

```
Node (taint: NoSchedule)

New Pod → ❌ cannot schedule → Pending
Existing Pod → ✔ continues running

```
```
NoSchedule → blocks new pods

```
---

### PreferNoSchedule

Kubernetes **tries to avoid scheduling pods**, but it is not strict.

Pods may still run if no better node is available.

---

### NoExecute

Pods that do not tolerate the taint will be **evicted from the node**.

```
Node (taint: NoExecute)

New Pod → ❌ cannot schedule
Existing Pod → ❌ removed from node

```

```
NoExecute → blocks new pods + removes existing pods

```
---

## 4. Example: Add a Taint to a Node

```bash
kubectl taint nodes node1 environment=production:NoSchedule
```

Meaning:

```
Pods cannot run on node1 unless they tolerate environment=production
```

---

## 5. Pod with Toleration

Pods must define **tolerations** to run on tainted nodes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "environment"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
```

Meaning:

```
This pod is allowed to run on nodes with the taint environment=production
```

---

## 6. Taints vs NodeSelector

| Feature          | NodeSelector          | Taints                           |
| ---------------- | --------------------- | -------------------------------- |
| Control Type     | Select nodes          | Block nodes                      |
| Scheduling Model | Pod chooses node      | Node rejects pods                |
| Use Case         | Target specific nodes | Protect nodes from unwanted pods |

---

## 7. Scheduling Model

```
Node
 └─ taint → environment=production:NoSchedule

Pod
 └─ toleration → environment=production
```

If toleration matches → pod can run.

If not → scheduling fails.

---

## 8. Interview Explanation

> Taints are applied to nodes to repel pods from being scheduled unless those pods have matching tolerations. This mechanism helps dedicate nodes for specific workloads and control scheduling behavior in Kubernetes clusters.
