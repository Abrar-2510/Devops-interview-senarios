# Kubernetes Node Affinity

## 1. What is Node Affinity?

Node Affinity is a more advanced way to control **which nodes a pod can run on**.

It works similar to **nodeSelector**, but provides more flexible rules.

Node Affinity uses **node labels** to decide where a pod should be scheduled.

---

## 2. Why Do We Use Node Affinity?

We use Node Affinity when we need **more complex scheduling rules**.

Examples:

* Run workloads only on GPU nodes
* Run databases on high-performance nodes
* Prefer SSD nodes but still allow others
* Control workloads across different environments

---

## 3. Types of Node Affinity

### RequiredDuringSchedulingIgnoredDuringExecution

This rule is **mandatory**.

If Kubernetes cannot find a node that matches the rule, the pod will stay **Pending**.

### PreferredDuringSchedulingIgnoredDuringExecution

This rule is **preferred but not required**.

Kubernetes will try to schedule the pod on matching nodes, but if none are available it can still run elsewhere.

---

## 4. Example: Required Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd
```

Meaning:

```
Pod will run only on nodes where disk=ssd
```

If no such node exists → Pod stays Pending.

---

## 5. Example: Preferred Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd
```

Meaning:

```
Prefer nodes with disk=ssd
But if none exist, schedule on other nodes
```

---

## 6. NodeSelector vs Node Affinity

| Feature          | NodeSelector     | Node Affinity              |
| ---------------- | ---------------- | -------------------------- |
| Complexity       | Simple           | Advanced rules             |
| Operators        | Exact match only | Supports In, NotIn, Exists |
| Preference rules | Not supported    | Supported                  |

---

## 7. Scheduling Model

```
Node
 └─ label → disk=ssd

Pod
 └─ nodeAffinity rule → disk In [ssd]
```

If the rule matches, Kubernetes schedules the pod on that node.

---

## 8. Interview Explanation

> Node Affinity is an advanced scheduling feature in Kubernetes that allows pods to run on nodes with specific labels using flexible matching rules such as In, NotIn, and Exists. It provides more control compared to nodeSelector.
