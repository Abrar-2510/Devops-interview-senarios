# 🚀 Kubernetes Interview Scenario: CrashLoopBackOff

This document explains the `CrashLoopBackOff` error in Kubernetes, including causes, reproducible test cases, tracking, and troubleshooting steps.

---

# 📌 What is CrashLoopBackOff?

`CrashLoopBackOff` means:

* The container **starts successfully**
* The application **crashes**
* Kubernetes **restarts it automatically**
* It crashes again
* Kubernetes applies **exponential backoff delay**

This results in a restart loop.

---

# 👀 Track It Live (Very Important in Interviews)

To monitor the restart behavior in real time:

```bash
kubectl get pods -w
```

The `-w` (watch) flag allows you to see:

* Container restarting
* Status changing to `CrashLoopBackOff`
* Restart count increasing

This is a strong troubleshooting technique to mention in interviews.

---

# 🔁 CrashLoopBackOff Internal Flow

1. Pod is created
2. Container starts
3. Application crashes
4. Kubelet restarts container
5. Crash happens again
6. Backoff delay increases (10s → 20s → 40s → up to ~5 minutes)

---

# 🛠️ Reproducible Test Cases (With YAML)

---

## 1️⃣ Wrong Command (Immediate Exit)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wrong-command
spec:
  containers:
    - name: app
      image: nginx
      command: ["/bin/false"]
```

Apply it:

```bash
kubectl apply -f wrong-command.yaml
kubectl get pods -w
```

### Why It Crashes

`/bin/false` exits immediately → container stops → restart loop.

---

## 2️⃣ Liveness Probe Misconfiguration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-liveness
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /wrongpath
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
```

Apply and track:

```bash
kubectl apply -f bad-liveness.yaml
kubectl get pods -w
```

### Why It Crashes

Probe checks invalid path → returns 404 → kubelet kills container → restart loop.

---

## 3️⃣ Out Of Memory (OOMKilled)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-test
spec:
  containers:
    - name: stress
      image: polinux/stress
      resources:
        limits:
          memory: "50Mi"
      args: ["--vm", "1", "--vm-bytes", "100M", "--vm-hang", "1"]
```

Apply and track:

```bash
kubectl apply -f oom-test.yaml
kubectl get pods -w
```

Check details:

```bash
kubectl describe pod oom-test
```

Look for:

```text
Reason: OOMKilled
```

---

## 4️⃣ Application Exit / Bug Simulation

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bug-app
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo Starting && sleep 3 && exit 1"]
```

Apply and watch:

```bash
kubectl apply -f bug-app.yaml
kubectl get pods -w
```

Container runs → exits with error → restarted repeatedly.

---

# 🔍 How To Troubleshoot CrashLoopBackOff

## Step 1: Watch Pod Status

```bash
kubectl get pods -w
```

---

## Step 2: Describe the Pod

```bash
kubectl describe pod <pod-name>
```

Check:

* Restart count
* Last State
* OOMKilled
* Probe failures

---

## Step 3: Check Logs

Current logs:

```bash
kubectl logs <pod-name>
```

Logs from previous crash:

```bash
kubectl logs <pod-name> --previous
```

---

# 🎯 Interview-Ready Explanation

> `CrashLoopBackOff` occurs when a container starts but crashes repeatedly.
> Kubernetes automatically restarts it and applies exponential backoff delay.
> Common causes include wrong command, liveness probe misconfiguration, low memory limits (OOMKilled), missing environment variables, or application bugs.
> I would monitor it using `kubectl get pods -w`, inspect it using `kubectl describe pod`, and check logs using `kubectl logs --previous`.

---

# 📊 Quick Comparison

| Feature                 | CrashLoopBackOff                             |
| ----------------------- | -------------------------------------------- |
| Container Starts?       | ✅ Yes                                        |
| Root Problem            | Application/runtime issue                    |
| Restart Policy Involved | Yes                                          |
| Key Debug Commands      | `get pods -w`, `describe`, `logs --previous` |

---

### CrashLoopBackOff Solutions

1. **تزويد initialDelaySeconds**

   * المشكلة: الـ liveness probe بيشتغل قبل ما التطبيق يجهز.
   * الحل:

   ```yaml
   livenessProbe:
     exec:
       command:
         - cat
         - /tmp/healthy
     initialDelaySeconds: 10
     periodSeconds: 5
   ```

2. **استخدام startupProbe**

   * مناسب للتطبيقات الثقيلة اللي بتاخد وقت لتبدأ.

   ```yaml
   startupProbe:
     exec:
       command:
         - cat
         - /tmp/healthy
     failureThreshold: 30
     periodSeconds: 2

   livenessProbe:
     exec:
       command:
         - cat
         - /tmp/healthy
     periodSeconds: 5
   ```

3. **تأكد إن الملف موجود**

   * الـ exec probe مش هينجح إلا لو الملف موجود.
   * مثال:

   ```bash
   touch /tmp/healthy
   ```

4. **استخدام HTTP Probe بدل exec**

   ```yaml
   livenessProbe:
     httpGet:
       path: /
       port: 8000
     initialDelaySeconds: 5
     periodSeconds: 5
   ```

5. **تزويد failureThreshold**

   * عشان الكونتينر ما يrestartش من أول غلطة.

   ```yaml
   failureThreshold: 5
   ```

---

### ليه استخدم readiness probe؟

* **liveness probe**: بيتأكد إن الكونتينر شغال ولا لأ. لو فشل، Kubernetes بيعمل restart.
* **readiness probe**: بيتأكد إن الكونتينر جاهز يستقبل ترافيك.
* استخدام readiness probe مفيد لأنك مش عايز Kubernetes يبعت طلبات للكونتينر قبل ما يكون جاهز، وده يمنع فشل الخدمة للـ users بدون ما تعمل restart للكونتينر.

# 🚀 Kubernetes Interview Scenario: CrashLoopBackOff with OOM

This document explains the `CrashLoopBackOff` error in Kubernetes, including causes, reproducible test cases, tracking, troubleshooting, and specifically **Out Of Memory (OOM) issues**.

---

# 📌 What is CrashLoopBackOff?

`CrashLoopBackOff` means:

* The container **starts successfully**
* The application **crashes**
* Kubernetes **restarts it automatically**
* It crashes again
* Kubernetes applies **exponential backoff delay**

This results in a restart loop.

---

# 💥 Out Of Memory (OOM) CrashLoopBackOff

**OOMKilled** happens when a container tries to use more memory than its **limit**. Kubernetes kills it and restarts the pod, which may lead to `CrashLoopBackOff` if repeated.

### Example Pod causing OOM

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-test
spec:
  containers:
    - name: stress
      image: polinux/stress
      resources:
        limits:
          memory: "50Mi"
      args: ["--vm", "1", "--vm-bytes", "100M", "--vm-hang", "1"]
```

**Why it crashes:**

* The container requests 100Mi RAM but the limit is only 50Mi
* Exceeds memory limit → OOMKilled → CrashLoopBackOff

### How to monitor

```bash
kubectl get pods -w
kubectl describe pod oom-test
kubectl logs oom-test --previous
```

Look for `Reason: OOMKilled` in the description.

---

# 🛠️ How to Prevent OOM CrashLoopBackOff

1. **Set realistic resource requests & limits**

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
```

2. **Use ResourceQuota on Namespace**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "4"
    limits.memory: "8Gi"
    pods: "10"
```

3. **Monitor memory usage**

```bash
kubectl top pod <pod-name>
kubectl top node
```

4. **Use startupProbe for heavy apps**

* Prevents liveness probe from killing container during initialization.

5. **Adjust livenessProbe failureThreshold and initialDelaySeconds**

* Allows container to stabilize before Kubernetes restarts it.

---

# 🎯 Interview-Ready Explanation

> CrashLoopBackOff due to OOM occurs when a container exceeds its memory limit. Kubernetes kills the container (OOMKilled) and restarts it repeatedly. Proper resource limits, monitoring, and probes prevent this scenario. Alw
