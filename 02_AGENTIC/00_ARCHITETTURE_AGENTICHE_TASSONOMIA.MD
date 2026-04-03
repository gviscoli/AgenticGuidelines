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


## I paradigmi dello sviluppo agentico
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