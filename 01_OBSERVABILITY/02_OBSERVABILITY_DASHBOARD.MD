# 📊 Metrics Documentation – Agent vs Skills Architecture

## Overview

Entrambi i progetti (`01_a2a_multi-agent_demo` e `02_skills_demo`) utilizzano OpenTelemetry per raccogliere metriche su:

* richieste
* errori
* latenza
* utilizzo token LLM

Le metriche sono esportate via OTLP → Prometheus → Grafana.

---

## 🔢 Metriche Disponibili

### 1. `agent_requests_total`

**Tipo:** Counter
**Descrizione:** Numero totale di richieste gestite

**Labels:**

* `agent`: nome agente/skill
* `status`: `ok` | `error`
* `tool` (opzionale)
* `kind` (opzionale: `wire` | `total`)

---

### 2. `agent_errors_total`

**Tipo:** Counter
**Descrizione:** Numero totale di errori

**Labels:**

* `agent`
* opzionale: `tool`

---

### 3. `agent_latency_seconds`

**Tipo:** Histogram
**Descrizione:** Tempo di risposta

**Labels:**

* `agent`
* `tool` (se chiamata interna)
* `kind`:

  * `wire` → latenza HTTP pura
  * `total` → latenza completa

---

### 4. `agent_tokens_total`

**Tipo:** Counter
**Descrizione:** Totale token consumati

---

### 5. `agent_input_tokens_total`

**Tipo:** Counter
**Descrizione:** Token input LLM

---

### 6. `agent_output_tokens_total`

**Tipo:** Counter
**Descrizione:** Token output LLM

---

## ⚠️ Limite Attuale

Le metriche NON includono:

* separazione per progetto (A2A vs Skills)
* identificatore architetturale

👉 Questo causa aggregazione dei dati tra i due sistemi.

---

## ✅ Raccomandazioni

Aggiungere label:

```python
{
  "agent": "...",
  "project": "a2a" | "skills"
}
```

Oppure usare:

```
service.name
```

nei filtri Grafana.

---

## 📊 Dashboard

### agents.json

* Focus: architettura multi-agent A2A
* Metriche per agente remoto

### skills.json

* Focus: esecuzione locale delle skills
* Metriche per funzione interna

⚠️ Attualmente entrambe leggono lo stesso dataset.

---

## 🎯 Obiettivo Corretto

Separare completamente:

| Dimensione | Valore              |
| ---------- | ------------------- |
| project    | a2a / skills        |
| agent      | meteo, mappe, etc   |
| tipo       | supervisor / worker |


---

*Documento aggiornato al 30 marzo 2026*
