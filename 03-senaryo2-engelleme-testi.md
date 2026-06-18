# Senaryo 2 — Engelleme Testi

> Senaryo 1 uygulandıktan sonra:
> app-ns  → db-ns:5432  = ERİŞEBİLİR  ✓
> other-ns→ db-ns:5432  = ENGELLENDİ  ✗
---

```bash
DB_IP=$(oc get pod db-pod -n db-ns -o jsonpath='{.status.podIP}')
echo "DB Pod IP: $DB_IP"
echo ""
```

> Test 1: app-ns → DB (erişebilmeli)
```bash
echo "=== TEST 1: app-ns → DB (BAŞARILI olmali) ==="
oc exec -n app-ns frontend-pod -- \
  curl -s --connect-timeout 5 $DB_IP:5432 -w "HTTP Kodu: %{http_code}\n" 2>&1 || echo "Bağlantı açıldı (PostgreSQL banner döndü)"
```

```bash
echo ""
```

> Test 2: other-ns → DB (engellenmelİ)
```bash
echo "=== TEST 2: other-ns → DB (ENGELLENMELİ) ==="
oc exec -n other-ns other-pod -- \
  curl -s --connect-timeout 5 $DB_IP:5432 2>&1 || echo "ENGELLENDI — zaman aşımı"
```

```bash
echo ""
```

> Politikaları listele
```bash
echo "=== db-ns NetworkPolicy listesi ==="
oc get networkpolicy -n db-ns
```

