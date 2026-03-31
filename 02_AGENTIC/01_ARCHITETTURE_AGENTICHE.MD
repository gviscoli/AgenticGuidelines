# Architetture Agentiche a Confronto
## Multi-Agent A2A vs Single-Agent con Skills dinamiche

---

## Architetture Agentiche

### Multi-Agent con comunicazione A2A
Un sistema **multi-agent** è un'architettura distribuita in cui ogni capacità del sistema è incapsulata in un agente autonomo e indipendente, ciascuno eseguito come processo separato con il proprio ciclo di vita, modello LLM e stack di dipendenze. Gli agenti comunicano tra loro attraverso un protocollo standardizzato di messaggistica — in questo caso **A2A (Agent-to-Agent)**, sviluppato da Google — che definisce un'interfaccia HTTP uniforme tramite la quale gli agenti si scoprono a vicenda, dichiarano le proprie capacità e si scambiano task e risposte.

**Struttura del sistema multi-agente (preso come riferimento il progetto `01_a2a_multi-agent_demo`):**

![A2A Multi-Agent Demo](/Images/ApplicationArchitecture/01_a2a_multi-agent_demo.png)

#### Funzionamento

Il supervisore non conosce a priori l'implementazione degli agenti. Al momento della chiamata, recupera dinamicamente la **AgentCard** dell'agente target — un documento JSON esposto all'endpoint `/.well-known/agent.json` — che dichiara nome, versione, capabilities, modalità di input/output e skill disponibili. Sulla base di questa auto-descrizione, il supervisore costruisce e invia un `SendMessageRequest` A2A. L'agente riceve la richiesta, la elabora autonomamente e restituisce un `Task` completato con gli artefatti di risposta.

La propagazione del contesto distribuito avviene tramite header **W3C TraceContext** (`traceparent`) iniettati in ogni chiamata HTTP, permettendo la ricostruzione della trace end-to-end in sistemi come Jaeger.

#### Sequence Diagram
Nella seguente immagine viene mostrato il sequence diagram della soluzione agentica multi-agente.

<div align="center">
      <img src="../Images/ApplicationArchitecture/01_a2a_multi-agent_demo_sequence.png" alt="01_a2a_multi-agent_demo sequence diagram" width="800">
</div>

#### Architettura Applicativa
Nella seguente immagine viene mostrata l'architettura applicativa di una soluzione agentica multi-agente.

<div align="center">
      <img src="../Images/ApplicationArchitecture/01_a2a_multi-agent_demo_apparch.png" alt="01_a2a_multi-agent_demo_apparch" width="800">
</div>

L'applicazione implementa il pattern Supervisor/Worker (gerarchico) con protocollo A2A (Agent-to-Agent di Google). Il flusso implementato prevede:

1. Il Client scopre il Supervisor via /.well-known/agent.json, poi invia richieste A2A

2. Il Supervisor (Voyager) su porta 8000 usa un ChatAgent con AWS Bedrock come LLM, che decide quale tool invocare (chiedi_meteo, chiedi_notizie, chiedi_valute, chiedi_mappe)

3. Ogni tool function esegue una chiamata A2A al rispettivo agent specializzato tramite ThreadPoolExecutor + asyncio.run

4. Ogni Agent specializzato (porte 8001-8004) ha il proprio ChatAgent con LLM (Ollama o Bedrock), tool interni che chiamano API esterne (ExchangeRate, Nominatim/OpenStreetMap, ecc.), e restituisce un Task con artifacts

5. Il Supervisor raccoglie i risultati, li passa a Bedrock per una sintesi finale, e restituisce la risposta al Client

6. OpenTelemetry è integrato a ogni livello per tracing distribuito e metriche

Tutte le comunicazioni tra il supervisor e gli agenti avvengono esclusivamente via A2A protocol, rendendo ogni agente indipendente e sostituibile.

### Single-Agent con Skills dinamiche

Un sistema **single-agent con skills dinamiche** è un'architettura monolitica in-process in cui un unico agente supervisore ingloba tutte le capacità del sistema sotto forma di **skill** — classi Python caricate dinamicamente a runtime. Non esistono processi separati né comunicazione di rete interna. Ogni skill è un modulo autonomo che incapsula il proprio LLM, i propri tool e la propria logica, ma viene istanziato e invocato direttamente all'interno del processo del supervisore.

La caratteristica distintiva di questa implementazione è la **SKILL.md discovery**: ogni skill si autodescrive tramite un file Markdown con frontmatter YAML che contiene metadati, istruzioni per il LLM supervisore e la documentazione del dominio. Il supervisore costruisce il proprio system prompt, la propria AgentCard e i propri tool **esclusivamente** leggendo questi file, senza alcuna configurazione hardcoded.

**Struttura del sistema mono-agente (preso come riferimento il progetto `02_skills_demo`):**

![Skills Demo](/Images/ApplicationArchitecture/02_skills_demo.png)

#### Funzionamento

Al boot, il `SkillLoader` scansiona la directory `skills/`, trova ogni sottocartella con un `SKILL.md`, ne fa il parsing del frontmatter YAML, importa dinamicamente la classe Python corrispondente via `importlib`, istanzia la skill e inietta in essa un `AgentMetrics` e un `Tracer` OpenTelemetry dedicati. Costruisce poi per ogni skill un `@ai_function` wrapper — con nome e docstring estratti dal SKILL.md — che fa da ponte tra il LLM supervisore e la skill.

Quando il supervisore riceve una richiesta, il LLM (Bedrock) analizza il testo e decide quale tool invocare. L'invocazione è una chiamata Python sincrona diretta: nessuna rete, nessuna serializzazione HTTP. La skill esegue il proprio agente Ollama in un thread separato (via `ThreadPoolExecutor`) per non bloccare l'event loop di uvicorn.

#### Sequence Diagram
Nella seguente immagine viene mostrato il sequence diagram della soluzione agentica mono-agente che utilizza l'approccio a skills.

<div align="center">
      <img src="../Images/ApplicationArchitecture/02_skills_demo_sequence.png" alt="02_skills_demo_sequence" width="800">
</div>

#### Architettura Applicativa
Nella seguente immagine viene mostrata l'architettura applicativa di una soluzione agentica mono-agente che utilizza l'approccio a skills.

<div align="center">
      <img src="../Images/ApplicationArchitecture/02_skills_demo_apparch.png" alt="02_skills_demo_apparch" width="800">
</div>

Approccio Skills (mono-agente) — un singolo processo Python su porta 9000 che contiene tutto:

1. Il SkillLoader al boot scansiona skills/*/SKILL.md, scopre le skill disponibili, e registra dinamicamente i tool nel ChatAgent del Supervisor

2. Ogni Skill (Meteo, Notizie, Valute, Mappe) è una classe Python che estende BaseSkill, con il proprio ChatAgent interno e i propri tool

3. Le skill vengono chiamate in-process (nessuna comunicazione di rete, nessun handoff HTTP) — il Supervisor invoca direttamente i metodi Python

Aggiungere una nuova skill = creare una cartella skills/nome/ con SKILL.md + classe. Zero modifiche al codice del supervisor. L'A2A protocol viene usato solo per la comunicazione Client → Supervisor

Rispetto alla versione A2A multi-agent: nessuna latenza di rete interna, un solo processo da gestire, nessun service discovery runtime. Il trade-off è che non puoi scalare le skill indipendentemente e un crash nel processo blocca tutto.

---

*Documento aggiornato al 30 marzo 2026*

