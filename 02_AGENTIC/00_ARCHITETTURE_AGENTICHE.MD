# Point of View: Architetture Agentiche

## Contesto e Stato dell'Arte
Lo sviluppo di sistemi agentici basati su Large Language Model (LLM) rappresenta oggi il paradigma dominante per l'automazione intelligente. Tutti i principali cloud provider e i laboratori di ricerca hanno pubblicato framework, guide architetturali e studi empirici sul tema. Questo documento sintetizza i principali punti di vista, offrendo un framework decisionale basato su evidenze.

Il passaggio dai semplici chatbot a sistemi agentici autonomi è stato rapido: **Amazon** riporta migliaia di agenti costruiti internamente dal 2025, **Gartner** prevede che il 40% delle applicazioni enterprise integrerà agenti task-specific entro fine 2026, e IBM stima che i workflow AI-enabled passeranno dal 3% al 25% entro fine 2025.

> **Fonti principali:**
> - [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
> - [OpenAI — A Practical Guide to Building Agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
> - [Microsoft — Single vs Multi-Agent Decision Framework](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/single-agent-multiple-agents)
> - [Google Cloud — Choose a Design Pattern for Your Agentic AI System](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system)
> - [Google DeepMind — Towards a Science of Scaling Agent Systems (arXiv:2512.08296)](https://arxiv.org/abs/2512.08296)
> - [AWS — Agentic AI Patterns and Workflows](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/introduction.html)

---

## Definizioni Fondamentali

### Workflow vs Agent

Anthropic introduce una distinzione architetturale cruciale tra **workflow** e **agent**:

- **Workflow**: sistemi in cui LLM e tool sono orchestrati attraverso percorsi di codice predefiniti (deterministici). Includono pattern come **prompt chaining**, **routing**, **parallelizzazione**.
- **Agent**: sistemi in cui l'LLM dirige dinamicamente i propri processi e l'uso dei tool, mantenendo il controllo su come raggiungere gli obiettivi.

Questa distinzione è condivisa da OpenAI nella propria guida pratica, dove si sottolinea che ogni approccio di orchestrazione richiede il concetto di "run loop" — un ciclo che consente all'agente di operare fino a una condizione di uscita.

> **Fonte:** [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
> **Fonte:** [OpenAI — Building Agents](https://developers.openai.com/tracks/building-agents)

### Mono-Agente (Single-Agent)

Un sistema mono-agente utilizza un singolo LLM potenziato con **tool** e/o **skills**, **retrieval** e **memoria** che gestisce l'intero task dall'inizio alla fine. Esiste **un unico loop di ragionamento**. L'agente può effettuare multiple chiamate al modello e iterare, ma la logica decisionale resta centralizzata in **un solo processo**.

### Multi-Agente (Multi-Agent)

Un sistema multi-agente distribuisce l'esecuzione del workflow tra più agenti coordinati, ognuno con un ruolo specializzato. Richiede pattern di orchestrazione espliciti — **supervisor/orchestrator**, **handoff peer-to-peer**, o **topologie ibride** — e meccanismi di comunicazione inter-agente.

---

## Il Punto di Vista dei Cloud Provider (oggi :smiley: )

### Anthropic — "Start Simple, Scale Intelligently"

La posizione di Anthropic è tra le più influenti nel settore. La guida **"Building Effective Agents"** di Erik Schluntz e Barry Zhang propone sei pattern composabili, da semplici workflow a sistemi agentici complessi:

1. **Prompt Chaining** — catena sequenziale con gate di qualità
2. **Routing** — classificazione e instradamento verso handler specializzati
3. **Parallelization** — esecuzione simultanea (sectioning o voting)
4. **Orchestrator-Workers** — decomposizione dinamica dei task
5. **Evaluator-Optimizer** — cicli iterativi di miglioramento
6. **Autonomous Agent** — loop completo con pianificazione autonoma

Il principio guida è chiaro: *"Start with simple prompts, optimize them with comprehensive evaluation, and add multi-step agentic systems only when simpler solutions fall short."* I sistemi multi-agente consumano circa 10-15x più token rispetto ai mono-agente, e richiedono mesi per essere messi a punto contro settimane per un agente singolo.

Anthropic raccomanda tre principi: mantenere la semplicità dell'architettura, investire in prompt engineering, e aggiungere complessità solo quando dimostrabilmente migliora i risultati.

> **Fonte:** [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
> **Fonte:** [Anthropic — Architecture Patterns and Implementation Frameworks (PDF)](https://resources.anthropic.com/hubfs/Building%20Effective%20AI%20Agents-%20Architecture%20Patterns%20and%20Implementation%20Frameworks.pdf)
> **Fonte:** [Anthropic Cookbook — Agents Patterns](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents)

### OpenAI — "Single Agent First, Multi-Agent When Needed"

OpenAI ha rilasciato l'Agents SDK (evoluzione del progetto Swarm) con primitive minimali: Agent, Handoff, Guardrails, Tracing. La guida pratica segue un approccio incrementale simile ad Anthropic.

Il framework supporta due pattern principali di orchestrazione multi-agente:

- **Manager Pattern (Agents as Tools)**: un agente centrale invoca sub-agenti specializzati come tool, mantenendo il controllo della conversazione. Preferito per trasparenza e parallelismo.
- **Handoff Pattern (Decentralized)**: agenti peer che si passano il controllo. Ottimale quando non serve supervisione centrale.

OpenAI raccomanda di partire con un singolo agente, espanderne le capability aggiungendo tool, e passare a multi-agente solo quando la complessità lo richiede. La guida enfatizza che un agente singolo con buone istruzioni e tool ben definiti può gestire scenari sorprendentemente complessi.

> **Fonte:** [OpenAI — A Practical Guide to Building Agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
> **Fonte:** [OpenAI Agents SDK — Orchestration](https://openai.github.io/openai-agents-python/multi_agent/)
> **Fonte:** [OpenAI Cookbook — Orchestrating Agents](https://cookbook.openai.com/examples/orchestrating_agents)

### Microsoft / Azure — Decision Tree Strutturato

Microsoft offre il framework decisionale più strutturato. Il Cloud Adoption Framework include un vero e proprio **decision tree** per la scelta tra architetture. Il messaggio è netto: i sistemi multi-agente vengono spesso scelti sulla base di assunzioni non verificate.

**Criteri per Multi-Agente (secondo Microsoft):**

- **Confini di sicurezza e compliance**: quando normative impongono isolamento dei dati tra domini
- **Team multipli coinvolti**: quando team distinti gestiscono aree di conoscenza separate e necessitano cicli di sviluppo indipendenti
- **Crescita futura pianificata**: quando la roadmap include funzionalità diverse, data source multiple, o business unit distinte (oltre 3-5 funzioni distinte)

**Costi del Multi-Agente evidenziati da Microsoft:**

- Ogni interazione tra agenti richiede design di protocolli, error handling, e sincronizzazione di stato
- Le superfici di sicurezza si espandono con credential management aggiuntivo
- La latenza si accumula ad ogni punto di handoff
- I costi crescono per il contesto ridondante processato da ciascun agente

Microsoft ha anche unificato AutoGen (ricerca) e Semantic Kernel (enterprise) nel nuovo **Microsoft Agent Framework** (public preview da ottobre 2025), che supporta sia **Agent Orchestration (LLM-driven)** che **Workflow Orchestration (deterministica)**.

> **Fonte:** [Microsoft — Single vs Multi-Agent Decision](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/single-agent-multiple-agents)
> **Fonte:** [Microsoft Agent Framework Overview](https://learn.microsoft.com/en-us/agent-framework/overview/)
> **Fonte:** [Introducing Microsoft Agent Framework — Azure Blog](https://azure.microsoft.com/en-us/blog/introducing-microsoft-agent-framework/)

### AWS — Lifecycle Progressivo e Resilienza

AWS adotta un approccio **lifecycle-oriented**. La guida per startup di AWS raccomanda esplicitamente:

- **Stage 1**: un singolo agente, un singolo outcome, stateless. Evitare framework multi-agente.
- **Stage 2**: aggiungere session memory, output strutturati, guardrails. "Still avoid swarms."
- **Stage 3+**: introdurre multi-agente solo quando la crescita lo richiede.

AWS fornisce anche guide dettagliate sulla resilienza dei sistemi agentici, identificando sette dimensioni architetturali (foundation model, orchestrazione, tool, memoria, guardrails, interfaccia utente, infrastruttura) e cinque modalità di fallimento comuni. Per la multi-agent orchestration, AWS propone un pattern Supervisor Agent con sub-agenti specializzati tramite **Amazon Bedrock AgentCore**.

> **Fonte:** [AWS — Build AI Agents That Scale](https://aws.amazon.com/startups/learn/build-ai-agents-that-scale-a-practical-lifecycle-for-startup-agent-architecture-?lang=en-US)
> **Fonte:** [AWS — Build Resilient Generative AI Agents](https://aws.amazon.com/blogs/architecture/build-resilient-generative-ai-agents/)
> **Fonte:** [AWS — AI Agents Best Practices with Bedrock AgentCore](https://aws.amazon.com/blogs/machine-learning/ai-agents-in-enterprises-best-practices-with-amazon-bedrock-agentcore/)
> **Fonte:** [AWS — Multi-Agent Orchestration Guidance](https://aws.amazon.com/solutions/guidance/multi-agent-orchestration-on-aws/)
> **Fonte:** [AWS — Agentic AI Patterns](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/introduction.html)


### Google Cloud — Pattern Catalogue con ADK

Google Cloud fornisce una guida architetturale completa tramite il Cloud Architecture Center, accompagnata dall'Agent Development Kit (ADK) — framework open-source ottimizzato per Gemini ma model-agnostic.

Google identifica pattern sia **single-agent** che **multi-agent**, con enfasi sulla scelta guidata dalle caratteristiche del workload: complessità del task, requisiti di latenza/performance, budget costi, e necessità di intervento umano.

Per i sistemi multi-agente, Google supporta interazione tramite shared session state, model-driven delegation (routing tra agenti), e invocazione esplicita (un agente chiama un altro come funzione). I protocolli chiave sono:

- **MCP (Model Context Protocol)**: standardizza connessione a sistemi esterni
- **A2A (Agent-to-Agent)**: protocollo per comunicazione inter-agente

Google enfatizza che i sistemi multi-agente richiedono valutazioni aggiuntive su sicurezza, affidabilità e costi rispetto ai mono-agente, e che ogni agente specializzato necessita di controlli di accesso precisi.

> **Fonte:** [Google Cloud — Choose a Design Pattern](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system)
> **Fonte:** [Google Cloud — Multi-Agent AI System Reference Architecture](https://docs.cloud.google.com/architecture/multiagent-ai-system)
> **Fonte:** [Google Cloud — Choose Architecture Components](https://docs.cloud.google.com/architecture/choose-agentic-ai-architecture-components)
> **Fonte:** [Google ADK Documentation](https://google.github.io/adk-docs/)
> **Fonte:** [Google Cloud Blog — Building Scalable AI Agents](https://cloud.google.com/blog/topics/partners/building-scalable-ai-agents-design-patterns-with-agent-engine-on-google-cloud)

---

## Evidenze dalla Ricerca Accademica

### Google DeepMind — "Towards a Science of Scaling Agent Systems"

Lo studio più rigoroso sul tema è il paper di **Yubin Kim et al**. (Google DeepMind/MIT, dicembre 2025), che ha testato **180 configurazioni** attraverso 5 architetture agentiche e 3 famiglie LLM (GPT, Gemini, Claude). I risultati principali:

**Amplificazione degli errori dipendente dalla topologia:**
- Sistemi multi-agente indipendenti (senza coordinazione) amplificano gli errori di **17.2x** rispetto al baseline mono-agente
- Sistemi con coordinazione centralizzata (orchestrator) contengono l'amplificazione a **4.4x** — l'orchestrator funziona come "validation bottleneck"

**Soglia di saturazione delle capability:**
- La coordinazione produce ritorni decrescenti o negativi quando il baseline mono-agente supera circa il **45%** di accuratezza
- Oltre 4 agenti, i guadagni di coordinazione si saturano

**Dipendenza dal tipo di task:**
- Coordinazione centralizzata migliora le performance dell'**80.8%** su task parallelizzabili (es. financial reasoning)
- Coordinazione decentralizzata eccelle nella web navigation (+9.2%)
- Per task di ragionamento sequenziale, **tutte le varianti multi-agente peggiorano le performance del 39-70%**

**Modello predittivo:** il team ha sviluppato un modello che usa proprietà misurabili del task (conteggio tool, decomponibilità) per predire l'architettura ottimale nell'87% delle configurazioni di test.

> **Fonte:** [arXiv:2512.08296 — Towards a Science of Scaling Agent Systems](https://arxiv.org/abs/2512.08296)
> **Fonte:** [Google Research Blog — When and Why Agent Systems Work](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)
> **Fonte:** [MIT Media Lab — Project Overview](https://www.media.mit.edu/projects/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/overview/)

### Cornell University — Vantaggi del Multi-Agente Coordinato

Uno studio della **Cornell University** ha mostrato che sistemi multi-agente sincronizzati raggiungono un tasso di successo del **42.68%** su task di pianificazione complessa, contro il **2.92%** di un singolo agente GPT-4. Questo dimostra che per specifiche classi di problemi — altamente parallelizzabili e cross-funzionali — il multi-agente è significativamente superiore.

> **Fonte citata in:** [Dataiku — Single-Agent vs Multi-Agent Systems](https://www.dataiku.com/stories/blog/single-agent-vs-multi-agent-systems)

### MAST — Multi-Agent Systems Failure Taxonomy

Lo studio MAST (marzo 2025) ha analizzato **1.642 tracce di esecuzione** attraverso 7 framework open-source multi-agente. I tassi di fallimento variavano dal **41% all'86.7%**, con la categoria di fallimento più grande rappresentata da **coordination breakdown (36.9%** di tutti i fallimenti).

> **Fonte citata in:** [Towards Data Science — The Multi-Agent Trap](https://towardsdatascience.com/the-multi-agent-trap/)

### UC Berkeley — Measuring Agents in Production

Il report di UC Berkeley (dicembre 2025), basato su 306 practitioner e 20 case study approfonditi (inclusi Accenture, Amazon, Google, IBM, Samsung), ha evidenziato che gli agenti in produzione non sanno usare efficacemente risorse extra di compute — l'85% del budget rimane inutilizzato.

> **Fonte citata in:** [ImagineX — Why Your Multi-Agent AI System Is Probably Making Things Worse](https://www.imaginexdigital.com/insights/why-your-multi-agent-ai-system-is-probably-making-things-worse)

---

## La Matematica Composta dell'Affidabilità

Un aspetto cruciale spesso sottovalutato è l'effetto composto dell'affidabilità. Se un singolo agente completa uno step con il 99% di affidabilità (eccellente), concatenare 10 step sequenziali produce:

**0.99¹⁰ = 90.4%** di affidabilità complessiva.

Con il 95% per step (ancora buono): **0.95¹⁰ = 59.9%**.

Ogni handoff tra agenti introduce un punto di potenziale fallimento. Modelli migliori spostano la curva ma non eliminano l'effetto composto. L'architettura determina se il sistema finisce nella zona del 60% o del 40% di affidabilità.

> **Fonte:** [Towards Data Science — The Multi-Agent Trap](https://towardsdatascience.com/the-multi-agent-trap/)

---

## Tabella Comparativa: Mono-Agente vs Multi-Agente (un primo assggio)

| Dimensione | Mono-Agente | Multi-Agente |
|---|---|---|
| **Complessità di sviluppo** | Bassa — settimane | Alta — mesi |
| **Costi operativi (token)** | Baseline | 10-15x il mono-agente |
| **Latenza** | Minima | Cumulativa ad ogni handoff |
| **Debugging/Audit** | Un singolo loop da tracciare | Multipli flussi da coordinare |
| **Sicurezza** | Superficie ridotta | Superfici espanse (credentials, transito dati) |
| **Scalabilità funzionale** | Limitata (~20 tool) | Potenzialmente illimitata |
| **Task parallelizzabili** | Non ottimale | +80.8% performance (centralizzato) |
| **Task sequenziali** | Ottimale | -39-70% performance |
| **Fault tolerance** | Single point of failure | Isolamento dei guasti (se ben progettato) |
| **Governance/Compliance** | Semplice | Richiede orchestrazione esplicita |

---

## Pattern Architetturali Multi-Agente

Quando il multi-agente è giustificato, esistono diverse topologie di coordinazione, ciascuna con trade-off specifici:

### Centralizzato (Supervisor/Orchestrator)

Un agente orchestratore gestisce la decomposizione dei task, delega ai worker, e sintetizza i risultati. È il pattern più raccomandato perché contiene l'amplificazione degli errori (4.4x vs 17.2x dei sistemi indipendenti). L'orchestrator funge da "validation bottleneck". Rischio: bottleneck se ogni decisione passa per un singolo agente.

### Decentralizzato (Handoff/Peer-to-Peer)

Agenti alla pari che si passano il controllo in base al contesto. Nessun supervisore. Eccelle nella web navigation (+9.2% rispetto al centralizzato). Rischio: perdita di contesto globale e coordinazione difficile.

### Ibrido

Combina supervisione centralizzata con esecuzione distribuita. Governance del mono-agente + capability del multi-agente. È dove si posizionano la maggior parte dei deployment enterprise seri. Framework come LangGraph, CrewAI, e il Microsoft Agent Framework supportano questo pattern.

### Sequenziale (Pipeline)

Agenti specializzati eseguiti in ordine lineare predefinito. Output di uno → input del successivo. Ottimale quando i passaggi sono ben definiti e prevedibili. Supportato nativamente da Google ADK.

---

## Framework e Tool dell'Ecosistema

| Framework | Provider | Caratteristiche Chiave |
|---|---|---|
| **Agents SDK** | OpenAI | Handoff, guardrails, tracing, Python-first |
| **Agent Framework** | Microsoft | Unione AutoGen + Semantic Kernel, workflow graph-based |
| **Amazon Bedrock AgentCore** | AWS | Runtime gestito, multi-agent collaboration, observability |
| **Agent Development Kit (ADK)** | Google | Multi-linguaggio, A2A, MCP, Agent Engine |
| **LangGraph** | LangChain | Grafi di stato, checkpointing, human-in-the-loop |
| **CrewAI** | Open Source | Role-based agents, process-driven |
| **Strands Agents SDK** | AWS (Open Source) | Reasoning loop leggero, tool orchestration |

> **Fonte:** [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
> **Fonte:** [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/)
> **Fonte:** [Google ADK](https://google.github.io/adk-docs/)
> **Fonte:** [AWS — Multi-Agent Orchestration](https://aws.amazon.com/solutions/guidance/multi-agent-orchestration-on-aws/)

---

## Protocolli di Comunicazione Inter-Agente

Due protocolli emergenti stanno standardizzando la comunicazione agentica:

- **MCP (Model Context Protocol)** — creato da Anthropic, standardizza come gli agenti si connettono a sistemi esterni, API, e data source. Riduce il tempo di integrazione fornendo un layer di comunicazione universale.

- **A2A (Agent-to-Agent Protocol)** — creato da Google, abilita la scoperta dinamica e la collaborazione tra agenti, anche se costruiti da team diversi con framework diversi. Elimina integrazioni point-to-point e silos funzionali.

Entrambi i protocolli sono supportati dal Microsoft Agent Framework, da Google ADK, e stanno diventando standard de facto.

> **Fonte:** [Tietoevry — Multi-Agent Systems with ADK, A2A, MCP](https://www.tietoevry.com/en/blog/2025/07/building-multi-agents-google-ai-services/)

---

## Framework Decisionale: Quando Usare Cosa

### Parti con un Mono-Agente quando:

- Il task è focalizzato e prevalentemente sequenziale
- Il contesto globale è importante (step 1 influenza step 5)
- Servono meno di ~20 tool
- Ci sono vincoli stretti di budget, latenza, o time-to-market
- Il baseline mono-agente supera il 45% di accuratezza sul task
- Un singolo team gestisce lo sviluppo

### Considera il Multi-Agente quando:

- I task sono genuinamente parallelizzabili e cross-funzionali
- Esistono confini di sicurezza/compliance che richiedono isolamento dei dati
- Team distinti gestiscono domini separati con cicli indipendenti
- La roadmap prevede >3-5 funzioni distinte
- Il ragionamento richiede profondità specializzata che un singolo agente non può fornire (es. fraud detection con analisi transazioni + storia cliente + aggregazione + validazione)

### Considera l'Ibrido quando:

- Serve affidabilità e governance (mono) con capability espanse (multi)
- Il workflow ha componenti sia deterministiche che aperte
- Si vuole autonomia limitata per i worker con escalation degli edge case

---

## Raccomandazioni Operative

1. **Validare le assunzioni prima di scegliere l'architettura.** Come sottolinea Microsoft, le architetture multi-agente sono spesso scelte sulla base di assunzioni non testate sulla complessità o le performance. Iniziare sempre con un prototipo mono-agente per stabilire un baseline.

2. **Rispettare la "regola del 45%".** Se il mono-agente già supera il 45% di accuratezza sul task, aggiungere agenti porterà ritorni decrescenti o negativi (DeepMind).

3. **Limitare a 4 agenti.** I guadagni di coordinazione si saturano oltre 4 agenti. Oltre quel numero, l'overhead di coordinazione consuma i benefici.

4. **Usare topologie strutturate, mai "bag of agents".** Sistemi non strutturati amplificano gli errori di 17.2x. Un orchestratore centralizzato riduce questo a 4.4x.

5. **Fare i conti dell'affidabilità composta.** Per ogni handoff, moltiplicare i tassi di fallimento. Questo esercizio da solo può cambiare la scelta architetturale.

6. **Investire in osservabilità.** Ogni agente necessita di tracciamento del ragionamento, monitoring delle invocazioni tool, e debugging delle interazioni. Senza osservabilità, il multi-agente diventa ingestibile.

7. **Progettare per evoluzione modulare.** Iniziare con mono-agente, progettare per la modularità (prompt centralizzati, tool con interfacce chiare, stato esternalizzato), evolvere verso multi-agente solo quando i dati lo giustificano.

---

## Conclusione

Il consenso emergente tra cloud provider, ricercatori e practitioner è sorprendentemente uniforme: **iniziare semplice, misurare tutto, aggiungere complessità solo quando dimostrabilmente migliora i risultati.**

Il mono-agente è sottovalutato: più veloce da sviluppare, più economico da operare, più facile da auditare, e la scelta giusta per più use case di quanto la maggior parte dei team oggi riconosca. Il multi-agente è genuinamente potente, ma solo quando l'orchestrazione è disciplinata, i ruoli sono definiti prima del deployment, gli handoff mappati esplicitamente, e il monitoring è continuo.

Come sintetizza efficacemente Anthropic: *"Success in the LLM space isn't about building the most sophisticated system. It's about building the right system for your needs."*

E come anticipano alcuni ricercatori, con il miglioramento dei modelli base, meno task si troveranno nella zona di bassa accuratezza dove agenti extra aggiungono valore reale. Col tempo, più problemi potranno essere risolti da un singolo modello end-to-end.

---

## Appendice: Indice delle Fonti

### Cloud Provider — Guide Ufficiali
| # | Fonte | URL |
|---|---|---|
| 1 | Anthropic — Building Effective Agents | https://www.anthropic.com/research/building-effective-agents |
| 2 | Anthropic — Architecture Patterns (PDF) | https://resources.anthropic.com/hubfs/Building%20Effective%20AI%20Agents-%20Architecture%20Patterns%20and%20Implementation%20Frameworks.pdf |
| 3 | OpenAI — A Practical Guide to Building Agents | https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/ |
| 4 | OpenAI — Agents SDK Orchestration | https://openai.github.io/openai-agents-python/multi_agent/ |
| 5 | Microsoft — Single vs Multi-Agent Decision | https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/single-agent-multiple-agents |
| 6 | Microsoft Agent Framework Overview | https://learn.microsoft.com/en-us/agent-framework/overview/ |
| 7 | Google Cloud — Design Pattern Selection | https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system |
| 8 | Google Cloud — Multi-Agent Architecture | https://docs.cloud.google.com/architecture/multiagent-ai-system |
| 9 | Google ADK | https://google.github.io/adk-docs/ |
| 10 | AWS — Build AI Agents That Scale | https://aws.amazon.com/startups/learn/build-ai-agents-that-scale-a-practical-lifecycle-for-startup-agent-architecture-?lang=en-US |
| 11 | AWS — Resilient AI Agents | https://aws.amazon.com/blogs/architecture/build-resilient-generative-ai-agents/ |
| 12 | AWS — Bedrock AgentCore Best Practices | https://aws.amazon.com/blogs/machine-learning/ai-agents-in-enterprises-best-practices-with-amazon-bedrock-agentcore/ |
| 13 | AWS — Multi-Agent Orchestration | https://aws.amazon.com/solutions/guidance/multi-agent-orchestration-on-aws/ |
| 14 | AWS — Agentic AI Patterns | https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/introduction.html |

### Ricerca Accademica
| # | Fonte | URL |
|---|---|---|
| 15 | Google DeepMind — Towards a Science of Scaling Agent Systems | https://arxiv.org/abs/2512.08296 |
| 16 | Google Research Blog — Scaling Agent Systems | https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/ |
| 17 | MIT Media Lab — Project Overview | https://www.media.mit.edu/projects/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/overview/ |

### Analisi e Commentary
| # | Fonte | URL |
|---|---|---|
| 18 | Towards Data Science — The Multi-Agent Trap | https://towardsdatascience.com/the-multi-agent-trap/ |
| 19 | Towards Data Science — Escaping the 17x Error Trap | https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/ |
| 20 | ImagineX — Why Your Multi-Agent System Is Making Things Worse | https://www.imaginexdigital.com/insights/why-your-multi-agent-ai-system-is-probably-making-things-worse |
| 21 | Dataiku — Single vs Multi-Agent Enterprise Tradeoffs | https://www.dataiku.com/stories/blog/single-agent-vs-multi-agent-systems |
| 22 | Decodingai — From 12 Agents to 1 | https://www.decodingai.com/p/from-12-agents-to-1-ai-agent-architecture-decision-guide |
| 23 | Neil Sahota — Single vs Multi-Agent AI | https://www.neilsahota.com/single-agent-vs-multi-agent-ai-the-real-cost-of-ai-agent-coordination/ |
| 24 | Galileo AI — Single vs Multi-Agent Systems | https://galileo.ai/blog/choosing-the-right-ai-agent-architecture-single-vs-multi-agent-systems |
| 25 | Devoteam — AWS AI Agentic Ecosystem | https://www.devoteam.com/expert-view/aws-ai-agentic-ecosystem/ |
| 26 | AWS — Evaluating AI Agents at Amazon | https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/ |

















# I quattro pilastri dell'architettura agentica
Lo sviluppo di sistemi agentici basati su LLM ha prodotto in pochi mesi un ecosistema ricco ma terminologicamente caotico. Termini come **"ReAct"**, **"function calling"**, **"supervisor pattern"** e **"MCP"**, **A2A** vengono spesso raggruppati sotto l'etichetta generica di "design pattern", ma in realtà appartengono a categorie concettuali distinte che operano a livelli diversi dello stack. Confonderli non è solo un problema accademico: porta a decisioni architetturali sbagliate, perché significa mescolare il come un agente ragiona con il come è strutturato il sistema o il come i componenti comunicano.

Per orientarsi in questo panorama, è utile scomporre il vocabolario dell'agentic AI in **quattro categorie fondamentali**, ciascuna con un ruolo preciso nella progettazione di un sistema agentico.

## La tassonomia per l'agentic AI

### Paradigmi di ragionamento 
Descrivono come l'agente pensa. Sono strategie cognitive, non strutture architetturali. **Chain-of-Thought**, **ReAct (Think-Act-Observe)**, **Tree of Thoughts**, **Reflexion** e **Graph of Thoughts** fanno parte di questa categoria. Sono più vicini ad algoritmi o strategie non sono dei design pattern. **ReAct** in particolare è un framework concettuale: definisce un loop (reason → act → observe) ma non prescrive come strutturare il sistema che lo implementa.

### Meccanismi di interazione
Descrivono come l'agente agisce. **Function Calling**, **Code-as-Action** e **Computer Use** sono meccanismi. Sono le **"interfacce"** tra l'agente e l'ambiente esterno. In termini di software engineering, sono più simili a protocolli o API contract che a design pattern.

### Design pattern
Nella definizione classica del software engineering (Gang of Four, 1994), un design pattern è una soluzione riusabile a un problema ricorrente in un contesto specifico. Ha quattro elementi: 
- un nome, 
- un problema che risolve, 
- una struttura della soluzione, 
- trade-off documentati. 

Il punto chiave è che un design pattern è prescrittivo a livello di struttura: ti dice come organizzare i componenti. Descrivono come strutturare il sistema. Qui rientrano i pattern multi-agente: **Supervisor/Worker**, **Pipeline/Sequential**, **Peer-to-Peer**, **Maker-Checker**, **Router**, **Subagents**, **Handoffs**. Questi sono design pattern nel senso classico: hanno un problema (coordinare agenti specializzati), una soluzione strutturale (un orchestratore delega a worker), e trade-off chiari (single point of failure sull'orchestratore vs. controllo centralizzato).

### Protocolli e standard
Descrivono come i componenti comunicano. **MCP**, **A2A** e **ACP** sono protocolli. Non sono design pattern, così come HTTP non è un design pattern anche se abilita pattern come REST.

## Perché la distinzione conta
Nella pratica, un sistema agentico di produzione combina elementi da tutte e quattro le categorie. Ad esempio, un Agente usa il **paradigma di ragionamento ReAct** (dentro ogni ChatAgent), il **meccanismo Function Calling** (per invocare i tool), il **design pattern Supervisor/Worker** (il supervisor delega agli agent specializzati), e il **protocollo A2A** (per la comunicazione di rete).


## La terminologia che si sta consolidando
La letteratura del 2026 converge su una terminologia a livelli che rispecchia questa distinzione. L'articolo di riferimento su arXiv (Agentic AI: Architectures, Taxonomies) propone una tassonomia a sei dimensioni: Core Components, Cognitive Architecture, Learning, Multi-Agent Systems, Environments, Evaluation. Il paper di ScienceDirect (Agentic AI: The Age of Reasoning) identifica cinque "key patterns": Tool Use, Reflection, ReAct, Planning e Multi-Agent Collaboration — ma usa il termine "pattern" in senso ampio, non nel senso GoF.

La proposta più pulita è quella che emerge dalla sintesi delle fonti: usare "agentic design patterns" per i pattern architetturali di coordinazione (Supervisor/Worker, Pipeline, ecc.) e "reasoning paradigms" per le strategie cognitive (ReAct, CoT, ToT). In questo modo si evita ambiguità.


I paradigmi dello sviluppo agentico
Lo sviluppo di sistemi agentici si è evoluto attraverso paradigmi progressivamente più sofisticati. Non si tratta di fasi che si sostituiscono, ma di livelli che si accumulano: i sistemi di produzione del 2026 ne combinano tipicamente quattro o cinque contemporaneamente.
1. Chain-of-Thought (CoT)
Il punto di partenza. L'LLM viene istruito a ragionare passo dopo passo prima di produrre una risposta. Non è ancora un "agente" in senso stretto — non agisce sul mondo esterno — ma è il prerequisito cognitivo per tutti i paradigmi successivi. Il limite principale è che il ragionamento avviene in isolamento: l'LLM non può verificare le sue ipotesi contro dati reali, il che porta a hallucination.
2. ReAct (Reasoning + Acting)
Il paradigma fondante dell'agentic AI, introdotto da Yao et al. nel 2023. L'agente entra in un loop iterativo di tre fasi: Think (ragiona sulla situazione), Act (esegue un'azione, ad esempio una chiamata API), Observe (interpreta il risultato). Il ciclo si ripete finché il task non è completato. Questo è il pattern più diffuso in produzione: LangChain, AutoGPT e la maggior parte dei framework agentici si basano su varianti di ReAct.
3. Function Calling / Tool Use
Introdotto da OpenAI nel giugno 2023, è il meccanismo attraverso cui l'LLM "usa le mani". Il modello viene fine-tuned per riconoscere quando una situazione richiede una chiamata a un tool esterno, e produce un JSON strutturato con nome della funzione e parametri. Oggi tutti i principali provider lo supportano (OpenAI, Anthropic, Google, Meta, IBM Granite). Il Function Calling è il meccanismo di basso livello; ReAct è il pattern di ragionamento che decide quando e perché chiamare un tool.
4. Reflection / Self-Correction
L'agente critica il proprio output prima di restituirlo. Funziona come un editor automatico integrato: dopo aver generato una risposta, un secondo passaggio (o un secondo agente) la valida contro criteri predefiniti. Il pattern Maker-Checker è la versione strutturata di questo paradigma. Riduce drasticamente le hallucination e migliora la qualità, al costo di latenza e token aggiuntivi.
5. Planning / Task Decomposition
L'agente scompone un obiettivo di alto livello in sotto-task strutturati prima di eseguire qualsiasi azione. Questo distingue un agente capace da un chatbot che reagisce ai prompt. Varianti avanzate includono Tree of Thoughts (esplora rami di ragionamento in parallelo) e LLM+P (il modello traduce l'obiettivo in PDDL e delega a un planner classico per la generazione ottimale della sequenza di azioni).
6. Code-as-Action
Invece di produrre JSON per function calling, l'agente scrive ed esegue codice Python direttamente. Questo amplia enormemente le capacità — qualsiasi computazione esprimibile in codice diventa un'azione possibile. Il paradigma CodeAct chiude il loop: se il codice fallisce, il traceback viene re-iniettato nel reasoning loop e l'agente corregge autonomamente. Smolagents di Hugging Face è costruito interamente su questo paradigma.
7. Multi-Agent Collaboration (MAC)
Agenti specializzati si coordinano per risolvere problemi complessi. Come ho dettagliato nell'analisi Excel, i pattern di collaborazione sono molteplici (supervisor/worker, peer-to-peer, pipeline, group chat, ecc.). Il 2026 segna il passaggio da loop di chat non strutturati a workflow graph controllabili con handoff espliciti.
8. Computer Use
L'agente interagisce con interfacce grafiche (browser, desktop) come farebbe un umano — clic, scroll, digitazione. Introdotto da Anthropic con Claude 3.5 Sonnet nel 2024 e da OpenAI con Operator nel 2025. Questo paradigma estende l'action space dagli API strutturati a qualsiasi software dotato di interfaccia.

## Panoramica su Tools, Skills e tecnologie emergenti
Questo è il punto dove la confusione è massima nel 2026. Facciamo chiarezza.
Tools (Function Calling)
Un tool è una funzione deterministica che un agente può invocare. Ha uno schema di input/output definito, fa sempre la stessa cosa per lo stesso input, e restituisce un risultato strutturato. Esempi: get_weather("Roma"), search_database(query), send_email(to, subject, body).
I tool sono il meccanismo di basso livello. Ogni LLM che supporta function calling può invocarli. Il "red flag" che cita la community: se il tuo "agente" produce sempre lo stesso output per lo stesso input, quello è un tool, non un agente.
Skills
Una skill è un livello di astrazione superiore al tool. Non è una singola funzione, ma un pacchetto di conoscenza e comportamento: un prompt specializzato, uno o più tool, e istruzioni su come e quando usarli. Il concetto è stato formalizzato con Claude Code nell'ottobre 2025 tramite file SKILL.md.
La differenza chiave rispetto ai tool è come interagiscono con il context window. Le skill usano progressive discovery: l'LLM vede inizialmente solo nome e descrizione (20-50 token ciascuna); solo quando decide di usare una skill, carica lo schema completo. Questo ha ridotto l'overhead di token dell'85% rispetto al caricamento completo di tutti i tool MCP.
Nel codice che hai condiviso prima (la versione Skills del tuo supervisor Voyager), ogni BaseSkill è esattamente questo: un SKILL.md per la discovery + una classe Python con il proprio ChatAgent interno e tool specifici.
MCP (Model Context Protocol)
MCP è il protocollo standard per la connessione agente → tool/dati. Creato da Anthropic, donato alla Linux Foundation nel dicembre 2025. A marzo 2026 ha superato i 97 milioni di download mensili degli SDK, con 10.000+ server attivi.
MCP risolve il problema N×M delle integrazioni: invece di N agenti × M servizi = N×M connettori custom, ogni agente implementa il client MCP una volta, ogni servizio implementa il server MCP una volta, e tutto comunica. È il "USB-C dell'AI".
Supportato da: Anthropic, OpenAI, Google, Microsoft, AWS, e praticamente ogni framework agentico.
A2A (Agent-to-Agent Protocol)
A2A è il protocollo standard per la comunicazione agente → agente. Creato da Google, donato anch'esso alla Linux Foundation. Ha raggiunto la v1.0 nel 2026 con supporto gRPC, Agent Cards firmati e multi-tenancy.
A2A risponde alla domanda: come fa un agente a scoprire, delegare e coordinarsi con un altro agente? Gli agenti pubblicano Agent Cards (JSON su /.well-known/agent-card.json) che dichiarano le proprie capabilities. Il tuo codice Voyager A2A usa esattamente questo pattern.
ACP (Agent Communication Protocol)
Il protocollo di IBM (progetto BeeAI). Più leggero di A2A: basato su REST puro, non richiede SDK specifici. Orientato alla governance enterprise e all'audit.
Lo stack emergente a 3 livelli
Il consenso nel 2026 è che questi protocolli non competono, ma si stratificano:

Layer 1 — WebMCP / AGENTS.md: come gli agenti accedono al web (file di istruzioni nei repository, sitemap agent-readable). Oltre 60.000 repository GitHub contengono già file AGENTS.md.
Layer 2 — MCP: come un agente accede a tool e dati (agent → tool). Il "sistema nervoso".
Layer 3 — A2A: come gli agenti collaborano tra loro (agent → agent). Il "sistema sociale".

Governance e standard
L'Agentic AI Foundation (AAIF), fondata sotto la Linux Foundation nel dicembre 2025 da Anthropic, Google, OpenAI e Block, governa sia MCP che A2A. A marzo 2026 conta 146 membri tra cui AWS, Microsoft, IBM, Salesforce, JPMorgan Chase. Il W3C sta lavorando a standard ufficiali per la comunicazione tra agenti, con specifiche attese per il 2026-2027.