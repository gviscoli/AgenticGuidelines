# Laboratorio Observability
Negli ultimi anni l’evoluzione delle architetture digitali — cloud‑native, data‑driven e sempre più **agentiche**, ha messo in evidenza un limite strutturale dei modelli tradizionali di monitoraggio: la difficoltà di comprendere i meccanismi causali che governano il comportamento dei sistemi, oltre la semplice osservazione dei sintomi. In questo scenario, l’**observability** non è più un tema esclusivamente operativo, ma una capacità strategica per garantire affidabilità, controllo e sostenibilità dell’evoluzione tecnologica.

Le organizzazioni operano oggi in **ecosistemi altamente distribuiti** e **interdipendenti**, in cui applicazioni, piattaforme dati e agenti software cooperano in modo dinamico per supportare processi critici di business. In tali contesti, caratterizzati da comportamenti emergenti, decisioni autonome e flussi non sempre deterministici, il monitoraggio tradizionale e la gestione reattiva degli incidenti risultano insufficienti. La capacità di ottenere una **visione integrata**, contestualizzata e continuativa dello stato dei sistemi, comprendendo non solo cosa accade, ma perché accade, diventa essenziale per preservare continuità operativa, qualità dei dati e fiducia nei processi decisionali.

Quando applicata come fondamento delle Intelligent Operations, l’observability abilita segnali affidabili, correlazione degli eventi, automazione e allineamento tra dimensione tecnica e impatto di business. Questo approccio consente di ridurre il rumore operativo, anticipare le criticità e trasformare informazioni frammentate in insight azionabili, permettendo interventi tempestivi e controllati anche in contesti ad alta criticità e time‑sensitivity.

È all’interno di questa visione che nasce il **Laboratorio di Observability**: uno spazio strutturato di sperimentazione progettato per tradurre principi architetturali in capacità concrete. Il laboratorio consente di validare pattern, strumenti e approcci operativi, osservando come telemetria, correlazione, visualizzazione e automazione cooperino lungo un ciclo continuo di osservazione, decisione e azione, accompagnando un percorso di maturazione da modelli reattivi verso approcci progressivamente più proattivi e predittivi.

In particolare, il laboratorio esplora il ruolo dell’observability come **abilitatore chiave dello sviluppo agentico**. In sistemi basati su agenti — capaci di prendere decisioni autonome, orchestrare azioni e interagire con altri sistemi — l’observability diventa una condizione imprescindibile di governabilità. Solo attraverso una visibilità profonda su contesto, decisioni, effetti e impatti è possibile introdurre autonomia in modo responsabile, mantenendo controllo, auditabilità e allineamento agli obiettivi di business.

In questa prospettiva, l’observability non è un accessorio dell’operatività né un semplice prerequisito tecnico: è la base strutturale su cui costruire soluzioni agentiche affidabili, scalabili e sostenibili, rendendo possibile l’evoluzione verso operazioni assistite e, selettivamente, autonome senza introdurre rischio sistemico.

# Architettura Observability Stack | Laboratorio
L'infrastruttura realizzata è basata sullo standard <b>OpenTelemetry (OTel)</b>, un framework open-source che permette di raccogliere, elaborare ed esportare dati di telemetria (metriche, tracce e log) in modo indipendente dal fornitore.

L'architettura segue un modello a "pipeline" dove i dati fluiscono dalle applicazioni agentiche fino alla visualizzazione finale su Grafana.

![Architettura](/Images/Architecture/Observability_HLA.jpg)

Questa rappresentazione mostra il percorso completo dei dati implemetata nel laboratorio per  la valutazione di diversi approcci nello sviluppo di <b>applicazioni agentiche</b>.

A Sinistra ci sono le <b>applicazioni agentiche</b> (A2A Multi-Agent e Skills In-Process) che generano metriche e tracce. Entrambe utilizzano il file <b>tracing.py</b> per inviare i dati tramite protocollo OTLP su gRPC (porta 4317).

Al Centro <b>OpenTelemetry Collector</b>: Il Collector riceve tutti i dati, li raggruppa tramite il Batch Processor e li smista attraverso due pipeline distinte:

- Le Tracce (Blue path) vengono inviate a Jaeger.

- Le Metriche (Green path) vengono esposte per Prometheus (con il prefisso multiagent_ che abbiamo configurato).

A Destra ci sono i componenti per lo <b>Storage</b> e la <b>Visualizzazione</b>: <b>Prometheus</b> effettua uno scraping periodico delle metriche dal Collector mentre <b>Grafana</b> interroga Prometheus e Jaeger per disegnare le dashboard configurate.

## Qui di seguito in dettaglio delle componenti principali

### OpenTelemetry Collector (otel-collector)
È il cuore pulsante dell'architettura. Invece di far inviare i dati dall'applicazione direttamente a <b>Prometheus</b> o <b>Jaeger</b>, l'applicazione li invia al Collector.

<b>Ruolo</b>: riceve i dati tramite protocollo <b>OTLP (OpenTelemetry Protocol)</b>, li aggrega, li filtra e li smista verso i database corretti. 

<b>Configurazione</b>: 
- Attualmente configurato sulla <b>porta 4317 (gRPC)</b>, viene usato classe Python Tracing per l'invio rapido delle metriche.
- Espone le metriche in formato Prometheus per essere lette dal server Prometheus.

Documentazione: [OpenTelemetry Collector Official Guide](https://opentelemetry.io/docs/specs/otlp/ "OpenTelemetry Collector Official Guide") 

### Jaeger
È un sistema open-source per il <b>distributed tracing</b> (tracciamento distribuito) utilizzato per monitorare e debuggare microservizi complessi.

<b>Ruolo</b>: permette di vedere il "viaggio" di una richiesta attraverso i vari agenti (specialmente utile nel progetto nelle architetture multi-agente). Se un agente chiama un altro tramite HTTP, Jaeger collega queste chiamate in un'unica traccia temporale.

<b>Configurazione</b>: 
- Il servizio Jaeger è raggiungibile sulla <b>porta 16686</b>.

Documentazione: [Jaeger Documentation](https://www.jaegertracing.io/docs/2.16/ "Jaeger Documentation") 


### Prometheus
È un sistema di monitoraggio e alerting open-source basato su serie temporali <b>Time Series Database</b> specializzato per le Metriche.

<b>Ruolo</b>: effettua lo scraping periodico (pull model) dell'OpenTelemetry Collector per scaricare le metriche definite nell'applicazione (nel caso specifico del laboratorio per la valutazione di applicazioni agentiche, dati sui token, il numero di richieste, le latenze e altro ancora).

Documentazione: [Prometheus Overview](https://prometheus.io/docs/introduction/overview/ "Prometheus Overview") 


### Grafana (grafana)
È un'interfaccia personalizzabile che organizza grafici e tabelle ("pannelli") per monitorare dati in tempo reale da varie fonti. Nel nostro caso è il livello di visualizzazione (Dashboard) che viene utilizzato per rappresentare il funzionamento delle applicazioni monitorate.

<b>Ruolo</b>: si collega a Prometheus come sorgente dati per creare le dashboard seguenti:

#### Dashboarda della soluzione multi-agente

![Dashboard applicazione agentica multi-agente](/Images/Dashboard/A2A_01.png)

#### Dashboard della soluzione mono-agente con skills

![Dashboard applicazione agentica mono-agente con skills](/Images/Dashboard/SKILLS_01.png)


Documentazione: [Grafana Documentation](https://grafana.com/docs// "Grafana Documentation") 


## Flusso dei Dati (Data Pipeline)
Il percorso che compie una metrica, per esempio <b>agent_input_tokens_total</b> è il seguente:

1. <b>Generazione</b>: L'applicazione Python usa la classe Tracing per registrare i token.

2. <b>Esportazione</b>: La libreria OpenTelemetry invia i dati al componente otel-collector sulla <b>porta 4317</b>.

3. <b>Elaborazione</b>: Il Collector riceve i dati, aggiunge i metadati e li mette a disposizione sulla <b>porta 8889</b>.

4. <b>Archiviazione</b>: Prometheus legge i dati dal Collector e li salva nel suo database interno.

5. <b>Visualizzazione</b>: Grafana esegue una query (PromQL) su Prometheus e disegna il grafico in tempo reale.

---

*Documento aggiornato al 30 marzo 2026*