# Kubernetes / OpenShift emptyDir (Temporary Disk) - Açıklama

## 📌 emptyDir Nedir?

`emptyDir`, Kubernetes/OpenShift içinde Pod’a özel oluşturulan **geçici (ephemeral) storage alanıdır**.

* Pod başlatıldığında oluşturulur
* Pod içindeki container’lar tarafından kullanılır
* Pod silindiğinde tamamen yok olur

👉 Yani kalıcı bir disk değildir, sadece “geçici çalışma alanı”dır.

---

## 🧠 emptyDir Nasıl Çalışır?

### 1. Pod Node’a atanır

Pod bir worker node üzerinde çalıştırılır.

### 2. Node üzerinde klasör oluşturulur

Kubernetes kubelet, node filesystem üzerinde bir dizin yaratır:

```
/var/lib/kubelet/pods/<pod-id>/volumes/kubernetes.io~empty-dir/<volume-name>
```

Bu dizin aslında emptyDir’in fiziksel karşılığıdır.

---

### 3. Container içine mount edilir

Bu dizin container içine bind edilir:

```
Node path  → Container path
emptyDir   → /tmp/data
```

Container için bu klasör normal bir disk gibi görünür.

---

### 4. Pod lifecycle

| Aksiyon           | Sonuç            |
| ----------------- | ---------------- |
| Pod oluşturulur   | emptyDir oluşur  |
| Container restart | emptyDir korunur |
| Pod silinir       | emptyDir silinir |

---

## 💾 emptyDir Disk midir?

Evet ama klasik anlamda değil.

emptyDir 3 şekilde olabilir:

### 1. Node filesystem (default)

```yaml
emptyDir: {}
```

* Node disk kullanır
* En yaygın kullanım

---

### 2. RAM (tmpfs)

```yaml
emptyDir:
  medium: Memory
```

* RAM üzerinde çalışır
* Çok hızlıdır
* Memory tüketir

---

### 3. Ephemeral storage limit

Kaynak kullanımı sınırlandırılabilir:

```yaml
resources:
  requests:
    ephemeral-storage: "1Gi"
  limits:
    ephemeral-storage: "2Gi"
```

---

## 🔄 emptyDir vs PVC

| Özellik   | emptyDir              | PVC                         |
| --------- | --------------------- | --------------------------- |
| Kalıcılık | ❌ Yok                 | ✔️ Var                      |
| Scope     | Pod bazlı             | Cluster / Node bağımsız     |
| Paylaşım  | Pod içi container’lar | Podlar arası                |
| Use case  | cache, temp, scratch  | database, log, data storage |

---

## 📦 Kullanım Senaryoları

### ✔️ Cache

* Dependency cache
* Build cache

### ✔️ Temporary file processing

* Upload işlemleri
* Dosya dönüşümleri

### ✔️ Multi-container paylaşım

* Sidecar pattern
* Log processing

---

## 🧩 Multi-container Örnek

```
Container A  → yazma (/data)
Container B  → okuma (/data)
                ↓
             emptyDir
```

---

## ⚡ Özet

* emptyDir = Pod’a özel geçici storage
* Node üzerinde klasör olarak oluşur
* Container içine mount edilir
* Pod silinince tamamen silinir
* PVC gibi kalıcı değildir

---

## 🚀 Basit Analogy

* PVC → Hard disk (kalıcı storage)
* emptyDir → Geçici çalışma alanı (scratch disk)
* ConfigMap → Read-only config dosyası
* Secret → Güvenli config

---

## 🎯 Sonuç

emptyDir, Kubernetes/OpenShift içinde:

* hızlı
* geçici
* Pod bazlı

bir storage çözümüdür ve genelde cache, temp file ve container içi veri paylaşımı için kullanılır.
