# Comparazione: Soluzioni Agentiche Mono-Agente vs Multi-Agente

| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| **ARCHITETTURA** | | |
| Definizione | Un singolo agente AI gestisce tutte le fasi: pianificazione, esecuzione, valutazione. Opera indipendentemente con la propria logica e tool. | Due o più agenti specializzati collaborano sotto un orchestratore. Ogni agente ha ruolo, competenze e tool dedicati. |
| Complessità Architetturale | Bassa: un prompt, un modello, tool opzionali. Nessuna comunicazione inter-agente. | Alta: richiede orchestrazione, protocolli comunicazione, gestione stato condiviso, retry e fallback. |
| Modello Mentale | Freelancer tuttofare: una persona gestisce tutto | Team specializzato: ricercatore, analista, scrittore, revisore, ciascuno con un ruolo preciso |
| Orchestrazione | Non necessaria (esecuzione lineare) | Necessaria: centralizzata (supervisor), decentralizzata (peer-to-peer), o ibrida |
| Gestione Stato | Contesto unificato in un'unica finestra | Stato condiviso tra agenti; richiede persistence, checkpointing, context propagation |

| **PERFORMANCE & SCALABILITÀ** | | |
| Latenza | Bassa: nessun handoff tra agenti, risposta diretta | Più alta: ogni handoff aggiunge latenza; ottimizzabile con parallelismo |
| Throughput | Limitato: elaborazione sequenziale, un task alla volta | Alto: agenti paralleli processano sub-task simultaneamente (3x più veloce secondo enterprise report) |
| Scalabilità | Verticale (modello più potente); non scala orizzontalmente | Orizzontale: aggiunta di agenti specialisti; scala con il volume dei task |
| Context Window | Rischio di overflow: un agente deve contenere tutto il contesto | Context isolation: ogni agente lavora solo con il contesto rilevante (67% meno token — LangChain benchmark) |

| **AFFIDABILITÀ & MANUTENZIONE** | | |
| Fault Tolerance | Single point of failure: se l'agente fallisce, tutto si ferma | Resiliente: un agente può fallire senza bloccare il sistema; retry, degradation, escalation |
| Debugging | Semplice: un flusso lineare, log unificato | Complesso: interazioni distribuite, necessita observability (tracing, LangSmith, ecc.) |
| Rischio Allucinazioni | Contenuto: un solo agente, un solo modello | Cascading hallucinations: Agent A allucinazione → Agent B la amplifica |
| Costo Operativo | Basso: meno token, meno API call, infra semplice | Più alto: token moltiplicati per agente; API call multiple; ma model tiering riduce 40-60% i costi |
| Manutenibilità | Facile: un prompt da gestire, un modello | Più complessa ma modulare: ogni agente è aggiornabile indipendentemente |

| **USE CASE IDEALI** | | |
| Chatbot / FAQ | ✓ Ideale: risposte dirette da knowledge base | Overkill per FAQ semplici |
| Code Generation | ✓ Buono con tool di esecuzione codice | ✓ Migliore per pipeline complesse (genera → testa → review) |
| Customer Support Complesso | OK per domande semplici | ✓ Ideale: refund + replacement + scheduling in una conversazione |
| Ricerca / Analisi Multi-Dominio | Limitato: un agente non copre tutti i domini | ✓ Eccellente: agenti specializzati per ogni fonte/dominio |
| Workflow Enterprise Multi-Step | Non adatto: perde contesto oltre 4-5 step | ✓ Progettato per questo: decomposizione task, parallelismo, escalation |
| Compliance / Security Boundaries | Non adatto: non può isolare dati per policy | ✓ Necessario: agenti con permessi diversi per data isolation (Microsoft CAF) |
| Diagnostica Medica | 74% accuracy (baseline) | 89% accuracy con agenti specialisti (Mayo Clinic case study 2025) |

| **COSTI & ROI** | | |
| Costo Sviluppo | Basso: ore/giorni per setup | Medio-Alto: settimane per orchestrazione, testing, observability |
| Costo Runtime (Token) | ~1x (baseline) | ~2-5x (moltiplicato per agenti); riducibile 40-60% con model tiering |
| ROI Potenziale | Buono per task specifici ad alto volume | Eccellente per workflow complessi: Klarna — $40M/anno risparmiati, resolution time da 11min a <2min |

| **RACCOMANDAZIONE** | | |
| Quando Partire con Single-Agent | ✓ Task ben definito, scope ristretto, contesto in un'unica finestra, MVP/PoC, budget limitato | — |
| Quando Passare a Multi-Agent | — | ✓ Task multi-dominio, >4 step, necessità di parallelismo, compliance boundaries, scaling enterprise |
| Best Practice | Partire SEMPRE con single-agent; validare ROI; evolvere a multi-agent solo quando necessario (Microsoft, Google, LangChain concordano) | Partire SEMPRE con single-agent; validare ROI; evolvere a multi-agent solo quando necessario (Microsoft, Google, LangChain concordano) |