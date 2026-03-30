# Comparazione: Soluzioni Agentiche Mono-Agente vs Multi-Agente

## Perché confrontare architetture mono-agente e multi-agente?
L'adozione di sistemi agentici basati su LLM sta accelerando rapidamente e di fronte a questa crescita, la prima decisione architetturale che ogni team deve affrontare è apparentemente semplice ma ha conseguenze profonde: è sufficiente un singolo agente, oppure servono più agenti specializzati che collaborano?

Non esiste una risposta universale. Un sistema mono-agente — un singolo LLM con i propri tool — è più veloce da costruire, più semplice da debuggare e perfettamente adeguato per task con scope ben definito. Un sistema multi-agente — dove agenti specializzati si dividono il lavoro sotto un orchestratore — diventa necessario quando crescono la complessità del dominio, i requisiti di parallelismo, o i vincoli di sicurezza e compliance.

Il rischio concreto è duplice: scegliere un'architettura multi-agente quando un singolo agente basterebbe significa moltiplicare costi, latenza e complessità operativa senza benefici reali. Ma restare su un singolo agente quando il task lo supera porta a perdita di contesto, allucinazioni e un sistema fragile che non scala.

Le tabelle che seguono mettono a confronto le due architetture su criteri concreti — dalla latenza al costo per token, dal fault tolerance ai use case reali — per fornire un framework decisionale basato su dati e non su hype. L'obiettivo non è stabilire quale approccio sia "migliore", ma aiutare a identificare quale sia quello giusto per il proprio caso specifico, seguendo il principio su cui Microsoft, Google e LangChain convergono: partire sempre con un singolo agente, validare il ROI, e evolvere verso un sistema multi-agente solo quando i requisiti lo richiedono.


## Architettura
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Definizione | Un singolo agente AI gestisce tutte le fasi: pianificazione, esecuzione, valutazione. Opera indipendentemente con la propria logica e tool. | Due o più agenti specializzati collaborano sotto un orchestratore. Ogni agente ha ruolo, competenze e tool dedicati. |
| Complessità Architetturale | Bassa: un prompt, un modello, tool opzionali. Nessuna comunicazione inter-agente. | Alta: richiede orchestrazione, protocolli comunicazione, gestione stato condiviso, retry e fallback. |
| Modello Mentale | Freelancer tuttofare: una persona gestisce tutto | Team specializzato: ricercatore, analista, scrittore, revisore, ciascuno con un ruolo preciso |
| Orchestrazione | Non necessaria (esecuzione lineare) | Necessaria: centralizzata (supervisor), decentralizzata (peer-to-peer), o ibrida |
| Gestione Stato | Contesto unificato in un'unica finestra | Stato condiviso tra agenti; richiede persistence, checkpointing, context propagation |

## Performance & Scalabilità
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Latenza | Bassa: nessun handoff tra agenti, risposta diretta | Più alta: ogni handoff aggiunge latenza; ottimizzabile con parallelismo |
| Throughput | Limitato: elaborazione sequenziale, un task alla volta | Alto: agenti paralleli processano sub-task simultaneamente (3x più veloce secondo enterprise report) |
| Scalabilità | Verticale (modello più potente); non scala orizzontalmente | Orizzontale: aggiunta di agenti specialisti; scala con il volume dei task |
| Context Window | Rischio di overflow: un agente deve contenere tutto il contesto | Context isolation: ogni agente lavora solo con il contesto rilevante (67% meno token — LangChain benchmark) |

## Affidabilità & Manutenzione
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Fault Tolerance | Single point of failure: se l'agente fallisce, tutto si ferma | Resiliente: un agente può fallire senza bloccare il sistema; retry, degradation, escalation |
| Debugging | Semplice: un flusso lineare, log unificato | Complesso: interazioni distribuite, necessita observability (tracing, LangSmith, ecc.) |
| Rischio Allucinazioni | Contenuto: un solo agente, un solo modello | Cascading hallucinations: Agent A allucinazione → Agent B la amplifica |
| Costo Operativo | Basso: meno token, meno API call, infra semplice | Più alto: token moltiplicati per agente; API call multiple; ma model tiering riduce 40-60% i costi |
| Manutenibilità | Facile: un prompt da gestire, un modello | Più complessa ma modulare: ogni agente è aggiornabile indipendentemente |

## Use Cases
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Chatbot / FAQ | ✓ Ideale: risposte dirette da knowledge base | Overkill per FAQ semplici |
| Code Generation | ✓ Buono con tool di esecuzione codice | ✓ Migliore per pipeline complesse (genera → testa → review) |
| Customer Support Complesso | OK per domande semplici | ✓ Ideale: refund + replacement + scheduling in una conversazione |
| Ricerca / Analisi Multi-Dominio | Limitato: un agente non copre tutti i domini | ✓ Eccellente: agenti specializzati per ogni fonte/dominio |
| Workflow Enterprise Multi-Step | Non adatto: perde contesto oltre 4-5 step | ✓ Progettato per questo: decomposizione task, parallelismo, escalation |
| Compliance / Security Boundaries | Non adatto: non può isolare dati per policy | ✓ Necessario: agenti con permessi diversi per data isolation (Microsoft CAF) |
| Diagnostica Medica | 74% accuracy (baseline) | 89% accuracy con agenti specialisti (Mayo Clinic case study 2025) |

## Costi & ROI
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Costo Sviluppo | Basso: ore/giorni per setup | Medio-Alto: settimane per orchestrazione, testing, observability |
| Costo Runtime (Token) | ~1x (baseline) | ~2-5x (moltiplicato per agenti); riducibile 40-60% con model tiering |
| ROI Potenziale | Buono per task specifici ad alto volume | Eccellente per workflow complessi: Klarna — $40M/anno risparmiati, resolution time da 11min a <2min |

## Raccomandazioni
| Criterio | Single-Agent (Mono-Agente) | Multi-Agent |
|---|---|---|
| Quando Partire con Single-Agent | ✓ Task ben definito, scope ristretto, contesto in un'unica finestra, MVP/PoC, budget limitato | — |
| Quando Passare a Multi-Agent | — | ✓ Task multi-dominio, >4 step, necessità di parallelismo, compliance boundaries, scaling enterprise |
| Best Practice | Partire SEMPRE con single-agent; validare ROI; evolvere a multi-agent solo quando necessario (Microsoft, Google, LangChain concordano) | Partire SEMPRE con single-agent; validare ROI; evolvere a multi-agent solo quando necessario (Microsoft, Google, LangChain concordano) |

## Pro e Contro

### Multi-Agent A2A — Pro e Contro

#### ✅ Pro

**Isolamento e fault tolerance reale.**
Ogni agente è un processo separato con il proprio spazio di memoria, dipendenze e ciclo di vita. Se `ValuteAgent` crasha — per un timeout verso ExchangeRate-API, per un bug, per esaurimento memoria — gli altri agenti continuano a funzionare indisturbati. Il supervisore riceve un errore da quel tool specifico e può gestirlo (risposta di fallback, retry, log) senza perdere l'intera sessione utente.

**Scalabilità indipendente per dominio.**
Se le domande sul meteo crescono di 10x rispetto alle domande sulle valute, si possono avviare 3 istanze di `MeteoAgent` dietro un load balancer senza toccare gli altri agenti. Ogni componente scala in base alla propria domanda reale, non alla domanda aggregata del sistema.

**Eterogeneità tecnologica.**
Ogni agente può essere scritto in un linguaggio diverso, usare un modello LLM diverso, deployare su hardware diverso (GPU dedicata per il modello più pesante, CPU economica per i placeholder). Il contratto è solo il protocollo A2A: l'implementazione interna è libera.

**Aggiornamenti senza downtime.**
Si può aggiornare `NotizieAgent` a una versione più recente — con un modello LLM migliore, tool reali al posto di placeholder — senza riavviare il supervisore o gli altri agenti. La nuova versione espone la stessa AgentCard, il supervisore continua a funzionare.

**Tracciabilità distribuita completa.**
La propagazione del `traceparent` W3C attraverso le chiamate HTTP permette di ricostruire in Jaeger la trace completa: si vede esattamente quanto tempo ha impiegato il supervisore, quanto ha impiegato la chiamata di rete, quanto ha impiegato l'agente a elaborare. Questo rende possibile individuare il collo di bottiglia esatto nel pipeline.

**Standard interoperabile.**
Il protocollo A2A è un standard emergente (Google, 2025). Un agente scritto con A2A può essere sostituito da un agente di un altro vendor o framework che implementa lo stesso protocollo, senza modificare il supervisore.

---

#### ❌ Contro

**Complessità operativa elevata.**
Per eseguire il sistema completo servono 5 processi, 5 porte di rete, 5 file di log da monitorare, 5 processi da riavviare in caso di crash. In sviluppo questo è scomodo; in produzione richiede un orchestratore (Kubernetes, Docker Compose, Supervisor process manager) e una strategia di health check per ogni agente.

**Overhead di latenza misurabile.**
Ogni delega dal supervisore a un agente richiede: risoluzione dell'AgentCard (HTTP GET), invio della richiesta (HTTP POST con serializzazione JSON), ricezione e deserializzazione della risposta. Nei test questo overhead ha prodotto una latenza media di ~5.27s contro ~3.50s delle Skills — un aumento del 51% sulla latenza assoluta, o equivalentemente: le Skills sono il 33% più veloci. Per richieste che delegano a più agenti (categoria "misto"), l'overhead si accumula.

**Ordine di avvio obbligatorio.**
Il supervisore deve trovare gli agenti già attivi quando si connette. Se un agente non è ancora in ascolto sulla propria porta, la prima chiamata fallisce. In ambienti di sviluppo questo richiede attenzione all'ordine di avvio dei terminali; in produzione richiede readiness probe e retry logic.

**Duplicazione del codice di boilerplate.**
Ogni agente replica la stessa struttura: `AgentExecutor`, estrazione del testo dal `context.message`, gestione del token usage, costruzione dell'`Artifact` di risposta. Nelle versioni attuali del progetto questo codice è incollato in tutti e 4 gli agenti, rendendo le modifiche trasversali (es. aggiungere un nuovo campo alla risposta) laboriose e soggette a dimenticanze.

**Debugging più difficile.**
Un bug in produzione richiede di correlare log da 5 processi separati, identificare la trace distribuita in Jaeger, e capire quale componente ha causato il problema. Senza un sistema di osservabilità ben configurato (come quello implementato con OpenTelemetry), il debugging diventa molto più complesso rispetto a un singolo processo con un unico stack trace.

---

### Single-Agent con Skills — Pro e Contro

#### ✅ Pro

**Semplicità operativa totale.**
Un solo processo da avviare, un solo log da monitorare, un solo health check, una sola porta di rete. Per eseguire l'intero sistema in sviluppo basta `python supervisor/supervisor.py`. In produzione basta un singolo container.

**Latenza inferiore.**
Eliminando il round-trip HTTP interno, la latenza media si riduce da ~5.27s a ~3.50s — un miglioramento del 33% misurato in condizioni identiche. Per categorie miste (che richiedono più deleghe) il vantaggio è ancora maggiore: da 7.41s a 4.25s (+74% di velocità). Questo guadagno è particolarmente rilevante per applicazioni interattive dove la latenza percepita dall'utente è critica.

**Estensibilità senza toccare il codice del supervisor.**
Aggiungere una nuova skill significa creare una cartella con `SKILL.md` e il file Python della skill. Al prossimo avvio il supervisore la scopre, la carica, aggiorna il proprio system prompt, aggiorna l'AgentCard e la rende disponibile come tool. Zero modifiche al codice esistente. Questo è il vantaggio principale dell'architettura rispetto a una versione naive di single-agent senza discovery.

**Iniezione di istruzioni dal dominio.**
Il corpo Markdown del `SKILL.md` viene iniettato nel system prompt del supervisore. Questo significa che le istruzioni su "quando usare questa skill", "quali tipi di domande gestisce", "quali sono i suoi limiti" sono scritte dai domain expert e mantenute insieme al codice della skill, non disperse nel codice del supervisore. La qualità del routing del LLM migliora con istruzioni precise e contestuali.

**Telemetria a due livelli senza infrastruttura distribuita.**
Grazie all'iniezione di `AgentMetrics` dedicati per ogni skill, le metriche mostrano sia la latenza end-to-end del supervisore sia la latenza interna di ogni singola skill — tutto da un unico processo, senza bisogno di propagazione di trace attraverso la rete.

**Debugging immediato.**
Un singolo stack trace Python contiene l'intera catena di chiamata: supervisor → skill → tool → risposta. Non serve correlare log da processi diversi o ricostruire trace distribuite.

---

#### ❌ Contro

**Fault isolation assente.**
Se una skill solleva un'eccezione non gestita — un timeout verso Nominatim, un bug nel codice della skill, un'eccezione nel modello Ollama — l'eccezione si propaga fino al supervisor e, se non correttamente gestita, può abbattere l'intera richiesta o in casi estremi il processo. Non c'è separazione di memoria o di fault domain tra skill diverse. Un bug catastrofico in `MappeSkill` (es. un loop infinito, un consumo anomalo di memoria) impatta tutte le altre skill.

**Scalabilità non granulare.**
Non è possibile scalare una singola skill in modo indipendente. Se `ValuteSkill` diventa il collo di bottiglia per un aumento del traffico sulle domande di valute, l'unica opzione è replicare l'intero processo del supervisore — con tutte le skill, incluse quelle non sotto pressione. Questo è inefficiente in termini di risorse.

**Eterogeneità tecnologica impossibile.**
Tutte le skill devono essere scritte in Python e devono girare nello stesso processo. Non è possibile avere una skill scritta in Go, una in Java, una che gira su una macchina diversa con GPU dedicata. Tutto il codice deve coesistere nello stesso ambiente Python con le stesse dipendenze, rischiando conflitti di versione.

**Il `ThreadPoolExecutor` introduce un vincolo nascosto.**
Poiché `BaseSkill.execute()` è un metodo sincrono chiamato da un tool sincrono, ma `ChatAgent.run()` è una coroutine, è necessario eseguire la coroutine in un thread separato per non bloccare l'event loop di uvicorn. Questo meccanismo funziona ma introduce un overhead di context switching e limita il numero di skill eseguibili in parallelo al pool size del `ThreadPoolExecutor`. In scenari di alta concorrenza questo può diventare un collo di bottiglia.

**Aggiornamenti richiedono riavvio completo.**
Per aggiornare la logica di una skill — anche solo la docstring nel `SKILL.md` — è necessario riavviare l'intero processo del supervisore, interrompendo tutte le sessioni attive. Nell'architettura A2A lo stesso aggiornamento richiederebbe solo il riavvio di un singolo agente.

**Il discovery è locale al filesystem.**
Le skill devono risiedere sullo stesso filesystem del supervisor. Non è possibile caricare una skill da un servizio remoto, da un repository esterno o da un pacchetto distribuito indipendentemente. Questo limita i pattern di distribuzione in ambienti cloud-native.

---

## Quando scegliere quale approccio

### Scegliere Multi-Agent A2A quando:

- Il sistema deve essere **fault-tolerant**: un dominio non disponibile non deve bloccare gli altri
- Ci sono **team separati** che gestiscono domini separati e devono poter deployare in modo indipendente
- Il sistema deve crescere verso **centinaia di agenti** specializzati gestiti da organizzazioni diverse
- I diversi domini richiedono **modelli LLM diversi**, hardware diverso o linguaggi di programmazione diversi
- Il contesto è **enterprise/multi-tenant** dove l'isolamento tra componenti è un requisito di sicurezza o compliance
- Serve **scalabilità orizzontale granulare** per gestire picchi di traffico asimmetrici tra domini

### Scegliere Single-Agent + Skills quando:

- Il sistema è gestito da **un unico team** e i domini sono relativamente stabili
- La **latenza percepita dall'utente** è critica e ogni millisecondo conta (applicazioni conversazionali real-time)
- Si vuole massimizzare la **semplicità operativa** minimizzando l'infrastruttura
- Le skill sono in numero limitato (fino a ~20) e possono coesistere nello stesso processo senza conflitti
- Il progetto è in fase di **prototipazione o MVP** e si vuole iterare rapidamente senza overhead infrastrutturale
- Le skill sono funzionalmente omogenee e richiedono lo stesso stack tecnologico (Python + stessa versione delle librerie)

---

## Fonti e Riferimenti Web Utilizzati

### Single-Agent vs Multi-Agent

| Fonte / Titolo | URL | Data / Note |
|---|---|---|
| Lyzr — Multi-Agent vs Single-Agent | https://www.lyzr.ai/blog/multi-agent-vs-single-agent/ | Mar 2026 |
| Analytics Vidhya — Single-Agent vs Multi-Agent Systems | https://www.analyticsvidhya.com/blog/2026/01/single-agent-vs-multi-agent-systems/ | Gen 2026 |
| Kubiya — Single Agent vs Multi Agent AI: Which Is Better? | https://www.kubiya.ai/blog/single-agent-vs-multi-agent-in-ai | Lug 2025 |
| Taskade — Single Agent vs Multi-Agent AI Teams (2026 Guide) | https://www.taskade.com/blog/single-agent-systems-versus-multi-agent-ai-teams | Feb 2026 |
| TechAhead — Single vs Multi-Agent AI: Which to Choose? | https://www.techaheadcorp.com/blog/single-vs-multi-agent-ai/ | Feb 2026 |
| Gino Marín — Single vs Multi-Agent AI: Complete Guide 2025 | https://www.ginomarin.com/articles/single-vs-multi-agent-ai | 2025 |
| Microsoft Learn — Choosing Single-Agent or Multi-Agent Systems (CAF) | https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/single-agent-multiple-agents | Dic 2025 |
| arXiv — Single-agent or Multi-agent Systems? Why Not Both? (2505.18286) | https://arxiv.org/abs/2505.18286 | Mag 2025 |
| Experro — Single Agent vs Multi Agent AI: Which to Choose in 2026? | https://www.experro.com/blog/single-agent-vs-multi-agent-ai/ | Mag 2025 |

### Pattern Multi-Agente & Orchestrazione

| Fonte / Titolo | URL | Data / Note |
|---|---|---|
| Microsoft Azure Architecture — AI Agent Orchestration Patterns | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns | Feb 2026 |
| Google Cloud — Choose a Design Pattern for Agentic AI | https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system | Ott 2025 |
| LangChain Blog — Choosing the Right Multi-Agent Architecture | https://blog.langchain.com/choosing-the-right-multi-agent-architecture/ | Gen 2026 |
| AI-AgentsPlus — Multi-Agent Orchestration Patterns 2026 | https://www.ai-agentsplus.com/blog/multi-agent-orchestration-patterns-2026 | Mar 2026 |
| Codebridge — Multi-Agent Systems & AI Orchestration Guide 2026 | https://www.codebridge.tech/articles/mastering-multi-agent-orchestration-coordination-is-the-new-scale-frontier | Feb 2026 |
| AgileSoftLabs — Multi-Agent AI Systems Enterprise Guide | https://www.agilesoftlabs.com/blog/2026/03/multi-agent-ai-systems-enterprise-guide | Mar 2026 |
| Chanl — The Multi-Agent Pattern That Actually Works in Production | https://www.chanl.ai/es/blog/multi-agent-orchestration-patterns-production-2026 | Mar 2026 |
| Swfte AI — Multi-Agent AI Systems for Enterprise 2026 | https://www.swfte.com/blog/multi-agent-ai-systems-enterprise | Dic 2025 |
| DEV.to — How to Build Multi-Agent Systems: Complete 2026 Guide | https://dev.to/eira-wexford/how-to-build-multi-agent-systems-complete-2026-guide-1io6 | Gen 2026 |

## Confronto Framework Agentici

| Fonte / Titolo | URL | Data / Note |
|---|---|---|
| GuruSup — Best Multi-Agent Frameworks 2026: LangGraph, CrewAI... | https://gurusup.com/blog/best-multi-agent-frameworks-2026 | Mar 2026 |
| Let's Data Science — AI Agent Frameworks 2026: LangGraph vs CrewAI & More | https://letsdatascience.com/blog/ai-agent-frameworks-compared | Mar 2026 |
| DEV.to — Top 7 AI Agent Frameworks 2026: Developer's Guide | https://dev.to/paxrel/top-7-ai-agent-frameworks-in-2026-a-developers-comparison-guide-hcm | Mar 2026 |
| Turing — A Detailed Comparison of Top 6 AI Agent Frameworks 2026 | https://www.turing.com/resources/ai-agent-frameworks | Feb 2026 |
| Softmax Data — Definitive Guide to Agentic Frameworks 2026 | https://blog.softmaxdata.com/definitive-guide-to-agentic-frameworks-in-2026-langgraph-crewai-ag2-openai-and-more/ | Feb 2026 |
| O-Mega — LangGraph vs CrewAI vs AutoGen: Top 10 AI Agent Frameworks | https://o-mega.ai/articles/langgraph-vs-crewai-vs-autogen-top-10-agent-frameworks-2026 | Gen 2026 |
| xPay — Top 10 AI Agent Frameworks 2026 | https://www.xpay.sh/blog/article/top-ai-agent-frameworks/ | Feb 2026 |
| StackOne — 120+ Agentic AI Tools Mapped [2026] | https://www.stackone.com/blog/ai-agent-tools-landscape-2026/ | Mar 2026 |
| OrchestrAI — Best AI Agent Frameworks 2026 (Ranked) | https://orchestrai.eu/blog/best-ai-agent-frameworks-2026 | Mar 2026 |
| Ampcome — 7 Best AI Agent Frameworks Compared (2026) | https://www.ampcome.com/post/top-7-ai-agent-frameworks-in-2025 | Ott 2025 (agg. 2026) |

---

*Documento aggiornato al 30 marzo 2026*
