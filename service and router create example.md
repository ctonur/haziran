# OpenShift Lab - Pod, Service ve Route Oluşturma

## Amaç

Bu laboratuvarda aşağıdaki işlemleri gerçekleştireceğiz:

* Bir Pod oluşturmak
* Pod'u Service ile yayınlamak
* Service'i Route ile dışarı açmak
* Label'ları incelemek
* Kaynakları label ile listelemek
* Oluşturulan kaynakları silmek

---

# 1. Pod YAML Dosyası

Dosya adı: `example-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
  labels:
    app: httpd
  namespace: onur
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: httpd
      image: image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest
      ports:
        - containerPort: 8080
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
```

---

# 2. Pod YAML'ını Dry Run ile Doğrulama

Pod'u oluşturmadan önce Kubernetes API tarafından kabul edilecek YAML çıktısını görmek için:

```bash
oc create -f example-pod.yaml \
  --dry-run=client \
  -o yaml
```

Açıklama:

* `--dry-run=client` kaynağı oluşturmadan doğrular.
* `-o yaml` oluşturulacak nesnenin YAML çıktısını gösterir.

---

# 3. Pod'u Oluşturma

```bash
oc apply -f example-pod.yaml
```

Pod durumunu kontrol et:

```bash
oc get pod -n onur
```

Detaylı inceleme:

```bash
oc describe pod example -n onur
```

---

# 4. Service YAML'ını Dry Run ile Oluşturma

Pod'u expose ederek Service oluşturacağız.

```bash
oc expose pod example \
  --port=8080 \
  --target-port=8080 \
  --name=httpd-svc \
  -n onur \
  --dry-run=client \
  -o yaml
```

Açıklama:

* `oc expose pod` Pod için Service oluşturur.
* `--port` Service portudur.
* `--target-port` Pod içindeki container portudur.
* `--name` Service ismidir.

---

# 5. Service'i Oluşturma

```bash
oc expose pod example \
  --port=8080 \
  --target-port=8080 \
  --name=httpd-svc \
  -n onur
```

Kontrol:

```bash
oc get svc -n onur
```

Detaylı inceleme:

```bash
oc describe svc httpd-svc -n onur
```

---

# 6. Route YAML'ını Dry Run ile Oluşturma

Service'i dış dünyaya açacak Route nesnesini oluşturmadan önce inceleyelim.

```bash
oc expose svc httpd-svc \
  -n onur \
  --dry-run=client \
  -o yaml
```

Açıklama:

* Route nesnesi oluşturulmadan YAML çıktısı gösterilir.
* Route Service'e yönlendirme yapar.

---

# 7. Route Oluşturma

```bash
oc expose svc httpd-svc -n onur
```

Kontrol:

```bash
oc get route -n onur
```

Detaylı inceleme:

```bash
oc describe route httpd-svc -n onur
```

---

# 8. Pod Label'larını Görüntüleme

Namespace içindeki tüm Pod label'larını göster:

```bash
oc get pod -n onur --show-labels
```

Beklenen label:

```text
app=httpd
```

---

# 9. Label ile Filtreleme

Sadece belirli label'a sahip Pod'ları listele:

```bash
oc get pod -n onur -l app=httpd
```

Açıklama:

* `-l` parametresi label selector kullanır.
* `app=httpd` etiketine sahip Pod'lar görüntülenir.

---

# 10. Tüm Kaynakları Label ile Listeleme

```bash
oc get all -n onur -l app=httpd
```

Bu komut aşağıdaki kaynakları gösterir:

* Pod
* Service
* Route
* ReplicaSet
* Deployment (varsa)

---

# 11. Service Selector Kontrolü

Service'in hangi Pod'lara trafik gönderdiğini incele:

```bash
oc get svc httpd-svc -n onur -o yaml
```

Selector kısmı:

```yaml
selector:
  app: httpd
```

Bu selector sayesinde Service ilgili Pod'u bulur.

---

# 12. Route Adresini Görüntüleme

```bash
oc get route httpd-svc -n onur
```

Örnek çıktı:

```text
httpd-svc-onur.apps.cluster.example.com
```

---

# 13. Route Testi

```bash
curl http://<route-hostname>
```

Örnek:

```bash
curl http://httpd-svc-onur.apps.cluster.example.com
```

Başarılı durumda HTTPD varsayılan sayfası döner.

---

# 14. Temizlik - Route Silme

```bash
oc delete route httpd-svc -n onur
```

Açıklama:

* Dış erişimi kaldırır.

---

# 15. Temizlik - Service Silme

```bash
oc delete svc httpd-svc -n onur
```

Açıklama:

* Service kaldırılır.
* Pod çalışmaya devam eder.

---

# 16. Temizlik - Pod Silme

```bash
oc delete pod example -n onur
```

Açıklama:

* Pod tamamen kaldırılır.

---

# 17. Son Kontrol

```bash
oc get all -n onur -l app=httpd
```

Beklenen çıktı:

```text
No resources found in onur namespace.
```

Bu çıktı laboratuvarın başarıyla temizlendiğini gösterir.
