# 🚦 Kubernetes لا يستقبل Traffic — دليل استكشاف الأخطاء

---

## 🧠 طبقات مسار الترافيك في Kubernetes

image_group{"layout":"carousel","aspect_ratio":"1:1","query":["kubernetes networking flow diagram external to pod","kubernetes ingress service pod flow diagram","kubernetes service endpoints diagram","kubernetes cluster networking layers diagram"],"num_per_query":1}

External → Ingress / LoadBalancer → Service → Endpoints → Pod → Container

أي مشكلة في أي طبقة من الطبقات السابقة قد تمنع وصول الترافيك.

---

# 🔍 أشهر الأسباب والحلول

---

## 1️⃣ Pod غير Ready

### افحص:

```bash
kubectl get pods
```

لو النتيجة:

```
0/1 Ready
```

### السبب المحتمل:

* Readiness Probe فاشلة
* التطبيق لم يكتمل تشغيله
* مشكلة اتصال بقاعدة البيانات

### الحل:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

## 2️⃣ Service لا يحتوي على Endpoints

### افحص:

```bash
kubectl get svc
kubectl get endpoints
```

لو ظهر:

```
<none>
```

### السبب:

عدم تطابق selector مع labels الخاصة بالـ Pods.

### تأكد من التطابق:

Service:

```yaml
selector:
  app: myapp
```

Pod:

```yaml
labels:
  app: myapp
```

---

## 3️⃣ نوع Service غير مناسب

| النوع        | الوصول من خارج الكلاستر |
| ------------ | ----------------------- |
| ClusterIP    | ❌                       |
| NodePort     | ✅                       |
| LoadBalancer | ✅                       |

### افحص:

```bash
kubectl get svc
```

لو النوع ClusterIP → لن يعمل من الخارج.

### الحل:

```yaml
type: NodePort
```

أو

```yaml
type: LoadBalancer
```

---

## 4️⃣ مشكلة في Ingress

image_group{"layout":"carousel","aspect_ratio":"1:1","query":["kubernetes ingress architecture diagram","kubernetes ingress controller flow","kubernetes ingress to service routing diagram","kubernetes ingress tls termination diagram"],"num_per_query":1}

### افحص:

```bash
kubectl get ingress
```

تأكد من:

* وجود Ingress Controller
* صحة DNS
* إعدادات TLS

---

## 5️⃣ خطأ في Ports

مثال:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

لو التطبيق يعمل على 5000 → لن يصل الترافيك.

تأكد من أن targetPort يطابق المنفذ داخل الحاوية.

---

## 6️⃣ CrashLoopBackOff

image_group{"layout":"carousel","aspect_ratio":"1:1","query":["kubernetes crashloopbackoff diagram","kubernetes liveness probe restart loop","kubernetes pod restart cycle","kubernetes kubelet restarting container"],"num_per_query":1}

### افحص:

```bash
kubectl describe pod <pod-name>
```

لو البود في حالة CrashLoopBackOff → لن يستقبل Traffic.

---

## 7️⃣ Firewall أو Security Group (في Cloud)

لو تستخدم LoadBalancer:

* تأكد من فتح المنفذ 80 أو 443
* راجع إعدادات Security Group

---

# 🏆 خطوات استكشاف الأخطاء في مقابلة عمل

1. افحص حالة الـ Pods
2. افحص Readiness
3. افحص Service
4. افحص Endpoints
5. افحص Ingress / LoadBalancer
6. افحص Ports
7. افحص الشبكة وFirewall

---

## أمر سريع يعطيك صورة كاملة

```bash
kubectl get all
```

---

💡 بهذه الطريقة تفكر كـ DevOps Engineer محترف عند مواجهة مشكلة عدم وصول الترافيك.
