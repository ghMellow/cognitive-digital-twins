# Manuale Utente — Cognitive Digital Twins for 5G Networks

**Autore:** Nicolò  
**Versione:** 1.0  
**Data:** Aprile 2026  
**Stato:** Prima release — fase sperimentale

---

## Panoramica del Progetto

Questo documento descrive l'ecosistema di repository sviluppato nell'ambito della tesi magistrale **"Cognitive Digital Twins for 5G Networks"**. Il progetto propone un'architettura di Digital Twin Cognitivo (CDT) per celle radio 5G, integrando una rappresentazione digitale in tempo reale con un framework multi-agente basato su LLM locali.

L'ecosistema è composto da quattro repository distinte, ciascuna con un ruolo preciso:

| Repository | Ruolo |
|---|---|
| `cognitive-digital-twins-thesis` | Teoria, manoscritto, wiki di ricerca |
| `cognitive-digital-twins-framework` | Architettura tecnica principale (in sviluppo) |
| `thesis-cdt-experiment-mas-memory` | Primo esperimento: benchmark multi-agente |
| `local-glm-ocr-pdf-to-markdown` | Tool di supporto: conversione PDF → Markdown |

---

## Architettura CDT — Visione d'Insieme

Il sistema CDT è strutturato su tre livelli:

```
┌──────────────────────────────────────────────────────────┐
│  LIVELLO 3 — COGNITIVO                                   │
│  LangGraph + LLM locali (Ollama) + Neo4j Knowledge Graph │
└──────────────────────────┬───────────────────────────────┘
                           │ bidirectional
┌──────────────────────────▼───────────────────────────────┐
│  LIVELLO 2 — DIGITAL TWIN                                │
│  Eclipse Ditto — rappresentazione digitale in tempo reale│
└──────────────────────────┬───────────────────────────────┘
                           │ bidirectional
┌──────────────────────────▼───────────────────────────────┐
│  LIVELLO 1 — FISICO SIMULATO                             │
│  Python — generatore metriche 3GPP                       │
└──────────────────────────────────────────────────────────┘
```

I tre contributi principali della tesi:

1. Architettura CDT completa con piena bidirezionalità tra i livelli
2. Framework di valutazione per agenti cognitivi in assenza di ground truth (contributo centrale)
3. Benchmark comparativo di modelli open-weight (scala 8B) su task 5G-specifici

---

## Repository 1 — Thesis Wiki (`cognitive-digital-twins-thesis`)

### Scopo

Repository della teoria e del manoscritto. Funge da **singola fonte di verità** per la struttura della tesi, i concetti teorici e lo stato della ricerca. Include la **LLM Wiki**: una knowledge base strutturata, mantenuta da un agente AI, usata per augmentare il ragionamento degli agenti con paper accademici e note di ricerca.

---

### Il pattern LLM Wiki — concetto base

La LLM Wiki è un'idea proposta da Andrej Karpathy nel 2026 (gist originale: [gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)).

L'idea di fondo è semplice: invece di far rileggere all'agente ogni volta i documenti sorgente (approccio RAG classico), si costruisce e si mantiene una wiki strutturata in Markdown. L'agente compila questa wiki incrementalmente man mano che nuove fonti vengono aggiunte. La conoscenza si **accumula** invece di essere ri-derivata da zero a ogni sessione.

Il sistema ha tre strati:

```
raw/        Sorgenti immutabili — PDF, trascrizioni, note (non toccate dall'agente)
wiki/       Pagine Markdown mantenute dall'agente — sintesi, concetti, cross-reference
CLAUDE.md   Schema operativo — istruzioni su come l'agente mantiene la wiki
```

La differenza con RAG è sostanziale: con RAG si recuperano frammenti grezzi a ogni query. Con la LLM Wiki, l'agente costruisce una comprensione sintetica e strutturata che cresce nel tempo — ogni nuova fonte rafforza i collegamenti esistenti invece di esistere in isolamento.

---

### Come creare una propria LLM Wiki da zero

Questi passaggi sono generici e replicabili con qualsiasi agente (Claude Code, GitHub Copilot, OpenCode, o altro strumento che legga file locali).

**Step 1 — Scegliere il frontend**

La wiki è una cartella di file Markdown plain — funziona con qualsiasi editor. [Obsidian](https://obsidian.md) è consigliato per la graph view, i backlink e la navigazione visiva, ma non è obbligatorio. VS Code, Typora, o qualsiasi editor di testo funzionano altrettanto bene. L'agente non dipende dal frontend scelto.

**Step 2 — Creare la struttura base delle cartelle**

```bash
mkdir -p my-wiki/raw
mkdir -p my-wiki/wiki
touch my-wiki/wiki/index.md
touch my-wiki/wiki/log.md
touch my-wiki/wiki/overview.md
touch my-wiki/CLAUDE.md
```

La struttura minima è questa:

```
my-wiki/
├── CLAUDE.md          ← istruzioni operative per l'agente (vedi Step 3)
├── raw/               ← materiali sorgente, sola lettura
│   ├── papers/
│   ├── notes/
│   └── ...
└── wiki/              ← pagine mantenute dall'agente
    ├── index.md       ← catalogo di tutte le pagine
    ├── log.md         ← storico operazioni, append-only
    ├── overview.md    ← sommario ad alto livello del dominio
    └── ...            ← pagine di concetti, entità, analisi
```

**Step 3 — Creare il CLAUDE.md (il file più importante)**

Il `CLAUDE.md` è il documento che trasforma un agente generico in un manutentore disciplinato della wiki. È qui che si definisce: come è organizzata la wiki, quali convenzioni seguire, cosa fare quando arriva una nuova fonte, come aggiornare le pagine esistenti.

Il punto di partenza consigliato è il **gist originale di Karpathy** (link sopra): contiene la struttura concettuale pulita e minimale. Va copiato, letto, e poi **personalizzato con l'aiuto dell'agente stesso** in base al proprio dominio specifico.

Il processo di personalizzazione è iterativo:

1. Copia il template base dal gist di Karpathy nel tuo `CLAUDE.md`
2. Apri il progetto con il tuo agente (es. Claude Code, Copilot in VS Code)
3. Di' all'agente: *"Leggi il CLAUDE.md, poi aiutami ad adattarlo per una wiki su [il tuo dominio]. Il mio obiettivo è [descrivi lo scopo]."*
4. L'agente proporrà sezioni, convenzioni e workflow specifici per il tuo caso
5. Rivedi, approva, e salva il risultato come nuovo `CLAUDE.md`

Il `CLAUDE.md` evolve nel tempo: man mano che si scopre cosa funziona e cosa no nel proprio workflow, si aggiorna collaborativamente con l'agente.

Un `CLAUDE.md` tipico contiene almeno queste sezioni:

```markdown
# Wiki Schema

## Struttura
[descrizione delle cartelle e del loro scopo]

## Convenzioni
[formato dei file, naming, frontmatter, cross-link]

## Workflow di ingestione
[passi da seguire quando arriva una nuova fonte]

## Workflow di aggiornamento
[come aggiornare pagine esistenti quando nuove info contraddicono o estendono]

## File speciali
[istruzioni specifiche per index.md, log.md, overview.md]

## Cosa NON fare
[vincoli espliciti — es. non modificare raw/, non cancellare log.md]
```

**Step 4 — Prima ingestione**

Una volta che `CLAUDE.md` è configurato, il workflow operativo di ogni sessione è:

1. Inserire il documento sorgente in `raw/` (PDF, MD, testo)
2. Dire all'agente: *"Segui le istruzioni in CLAUDE.md e ingerisci [nome file]"*
3. L'agente legge il sorgente, crea o aggiorna le pagine wiki, aggiorna `index.md` e `log.md`
4. Revisionare le modifiche prima di committare

**Step 5 — Uso della wiki nelle sessioni**

Una volta costruita, la wiki viene usata come contesto di partenza per ogni sessione. Invece di rileggere tutti i sorgenti, l'agente legge `index.md` e `overview.md` (pochi token) e recupera le pagine specifiche solo quando necessario.

---

### La LLM Wiki in questo progetto

La wiki della tesi segue esattamente questo pattern, adattato al dominio CDT/5G:

```
raw/                    Materiali sorgente (sola lettura)
  papers/               Paper con analisi pre-processata
  calls/                Trascrizioni di riunioni e discussioni
  project/              Proposta di tesi, feedback, note di ricerca

wiki/                   Knowledge base (manutenzione attiva)
  scaffolding-tesi.md   Struttura centrale dell'argomento di tesi
  glossary.md           Terminologia canonica e definizioni
  index.md              Navigazione e tracking dello stato
  log.md                Storico operazioni (append-only)
  overview.md           Sommario ad alto livello della tesi
  sources/              Sintesi dei documenti sorgente
  theoretical-concepts/ Concetti teorici e framework
```

**`scaffolding-tesi.md`** è il documento centrale specifico per questa tesi: contiene la domanda di ricerca, le ipotesi, l'outline dei capitoli, le tensioni aperte e i gap identificati nella letteratura. Va letto prima di qualsiasi operazione sulla wiki — è l'equivalente di un `overview.md` specializzato per seguire l'evoluzione dell'argomento di tesi.

**`glossary.md`** definisce la terminologia canonica CDT e 5G usata in tutto il progetto.

**`index.md`** è il catalogo completo di tutte le pagine wiki con tracking del progresso.

**`log.md`** è il registro cronologico di tutte le operazioni — append-only, non va modificato retroattivamente.

L'agente usato per la manutenzione è **GitHub Copilot in VS Code**, ma il sistema è agente-agnostico: qualsiasi strumento che legga file locali (Claude Code, OpenCode, Cursor, ecc.) funziona allo stesso modo. I processi operativi completi sono documentati in `CLAUDE.md` e `SKILL-thesis-ingest.md` nella repository.

### Skill PDF → Markdown

La wiki include una skill personalizzata per l'ingestione di PDF complessi (con grafici, tabelle, figure) tramite il tool `local-glm-ocr-pdf-to-markdown` (descritto sotto). La skill istruisce l'agente a chiamare l'endpoint locale esposto dal tool, ricevere il Markdown strutturato e inserirlo nella pipeline di ingestione standard.

---

## Repository 2 — Framework (`cognitive-digital-twins-framework`)

> **Stato:** In sviluppo — non ancora rilasciato.

Conterrà il motore tecnico principale del progetto: simulatore di metriche 5G, configurazioni Eclipse Ditto, schema Neo4j, e orchestrazione multi-agente LangGraph.

**Stack tecnologico previsto:** Python, LangGraph, Neo4j, Eclipse Ditto, LLM locali via Ollama (Llama 3.1).

---

## Repository 3 — Esperimento MAS Memory (`thesis-cdt-experiment-mas-memory`)

### Scopo

Primo esperimento controllato del progetto. Confronta il comportamento di agenti LLM con ruoli diversi (esperto vs beginner) su task matematici e di ragionamento 5G, variando l'architettura multi-agente.

### Prerequisiti

**Ollama** installato e in esecuzione con i modelli richiesti:

```bash
ollama serve
ollama pull qwen3:9b
ollama pull deepseek-r1:7b
```

Per verificare che i modelli siano disponibili:

```bash
ollama list
```

**Python 3.11+** e **Poetry** per la gestione dell'ambiente virtuale.

### Installazione

```bash
# Clona la repository
git clone https://github.com/ghMellow/thesis-cdt-experiment-mas-memory
cd thesis-cdt-experiment-mas-memory

# Installa le dipendenze con Poetry
poetry install

# Attiva l'ambiente (opzionale, per lavorare interattivamente)
poetry shell
```

In alternativa, senza Poetry:

```bash
# Esporta requirements da Poetry (una tantum)
poetry export -f requirements.txt --output requirements.txt

# Crea e attiva virtual environment
python -m venv .venv
source .venv/bin/activate      # Linux/macOS
.venv\Scripts\activate         # Windows

pip install -r requirements.txt
```

### Struttura del progetto

```
experiment/
├── docs/
│   ├── setup.md                  # Documentazione tecnica completa
│   └── tasks/
│       ├── task1_math_int.md         # Task matematico intero
│       ├── task1_math_int_sol.md     # Ground truth (solo checker)
│       ├── task2_math_real.md        # Task matematico reale
│       ├── task2_math_real_sol.md    # Ground truth con tolleranza
│       ├── task3_anomaly.md          # Classificazione anomalia 5G
│       ├── task3_anomaly_sol.md      # Rubrica per judge LLM
│       ├── task4_rootcause.md        # Root cause analysis 5G
│       └── task4_rootcause_sol.md    # Rubrica per judge LLM
├── agents/
│   ├── agent_runner.py           # Logica agente (esperto/beginner)
│   ├── judge_agent.py            # Agente giudice indipendente
│   └── prompts.py                # System prompt centralizzati
├── core/
│   ├── checker.py                # Confronto GT per task matematici
│   ├── loop_controller.py        # Gestione tentativi max
│   └── result_writer.py          # Salvataggio JSON risultati
├── results/                      # Output esperimenti (generato a runtime)
├── main.py                       # Entry point
└── config.py                     # Configurazione globale
```

### Setup esperimenti (1A vs 1B)

**Esperimento 1A — controllo:** esperto e beginner usano lo stesso modello (Qwen3 9B). Isola l'effetto del ruolo eliminando la variabile modello.

**Esperimento 1B — confronto:** esperto usa Qwen3 9B, beginner usa DeepSeek-R1 7B. Confronta l'effetto combinato di ruolo e modello diverso.

| Agente | Esperimento 1A | Esperimento 1B |
|---|---|---|
| Esperto (10 anni) | Qwen3 9B | Qwen3 9B |
| Beginner (1 anno) | Qwen3 9B | DeepSeek-R1 7B |

Entrambi gli esperimenti usano gli stessi 4 task e temperatura 0 per garantire determinismo.

### Task disponibili

**Task 1 — Matematico intero:** problema aritmetico su connessioni di antenna 5G. Verifica programmatica con confronto esatto.

**Task 2 — Matematico reale:** calcolo di media e deviazione standard campionaria su misure di throughput. Verifica programmatica con tolleranza ±0.5.

**Task 3 — Classificazione anomalia:** dato un set di KPI di un nodo 5G fuori soglia, classificare come `NORMALE`, `ANOMALIA_LIEVE` o `ANOMALIA_CRITICA`. Verifica tramite LLM judge con rubrica a punteggio.

**Task 4 — Root cause analysis:** identificare la causa probabile di un degrado su antenna 5G e proporre i primi 2 step diagnostici. Verifica tramite LLM judge con rubrica a punteggio.

### Formato output degli agenti

Ogni agente risponde esclusivamente in JSON strutturato:

```json
{
  "answer": "...",
  "reasoning": "...",
  "confidence": 0.0
}
```

Il campo `reasoning` contiene la catena di ragionamento esplicita. Il campo `confidence` è una stima soggettiva dell'agente (0.0–1.0) usata per valutare la calibrazione.

### Meccanismo di verifica e loop

Per i **task matematici**, una funzione Python confronta la risposta con la ground truth (caricata dal file `_sol.md`) senza mai esporla all'agente. Se la risposta è sbagliata, l'agente riceve solo il segnale `"sbagliato, riprova"` e itera fino a un massimo di `MAX_RETRIES` tentativi (default: 3).

Per i **task testuali**, un **agente judge separato e indipendente** valuta la risposta secondo la rubrica contenuta nel file `_sol.md`. Il judge non riceve la ground truth esplicita — riceve scenario originale, risposta dell'agente e rubrica. Anche qui il loop itera fino a `MAX_RETRIES` se il punteggio è insufficiente.

La ground truth non viene mai iniettata nel contesto dell'agente valutato, eliminando il rischio di bias.

### Esecuzione

```bash
# Esegue tutti gli esperimenti e tutti i ruoli (default)
poetry run python main.py

# Solo esperimento 1A
poetry run python main.py --experiment 1A

# Solo un ruolo e un task specifico
poetry run python main.py --experiment 1A --role expert --task task1_math_int

# Personalizza timeout e ripetizioni
poetry run python main.py --task-timeout 600 --repetitions 1
```

Parametri disponibili:

| Flag | Valori | Default |
|---|---|---|
| `--experiment` | `1A`, `1B`, `all` | `all` |
| `--role` | `expert`, `beginner`, `all` | `all` |
| `--task` | ID task (ripetibile) | tutti |
| `--repetitions` | intero | 3 |
| `--task-timeout` | secondi | 600 |

### Risultati e output

I risultati vengono salvati in `results/` con questa struttura:

```
results/
├── 1A/
│   ├── expert/
│   │   ├── task1_math_int_rep1.json
│   │   ├── task1_math_int_rep1_solution.json
│   │   └── ...
│   └── beginner/
├── 1B/
│   └── ...
└── evaluation/
    ├── scores_1A.md
    ├── scores_1B.md
    └── comparison.md
```

Ogni file `_repN.json` contiene:

```json
{
  "started_at": "...",
  "finished_at": "...",
  "elapsed_seconds": 0.0,
  "attempts": 1,
  "verdict": "correct",
  "final_answer": { ... },
  "history": [ ... ]
}
```

Il campo `history` registra tutti i tentativi, permettendo di analizzare come l'agente corregge il ragionamento tra un tentativo e l'altro.

### Stop, resume e skip

Il runner è progettato per essere interrompibile in sicurezza. A ogni restart, verifica quali file JSON esistono già in `results/` e salta automaticamente le ripetizioni già completate.

Esempio: con `--repetitions 3`, se `task1_math_int_rep1.json` e `rep2.json` esistono ma `rep3.json` manca, solo `rep3` verrà eseguita.

### Troubleshooting

| Problema | Soluzione |
|---|---|
| `model 'qwen3:9b' not found` | `ollama pull qwen3:9b` |
| Endpoint non raggiungibile | `ollama serve` |
| Esecuzione molto lenta | Iniziare con `--repetitions 1` e un solo task |
| Timeout frequenti | Aumentare `--task-timeout` e verificare che `OLLAMA_TIMEOUT_SECONDS` in `config.py` sia ≥ task_timeout |

---

## Repository 4 — PDF to Markdown (`local-glm-ocr-pdf-to-markdown`)

### Scopo

Tool locale per convertire PDF complessi (con grafici, tabelle, figure) in Markdown strutturato. Usa il modello multimodale `glm-ocr:latest` via Ollama, che effettua screenshot delle pagine e ne estrae il contenuto visivamente — superando i limiti dei parser testuali tradizionali su documenti con layout complesso.

È stato sviluppato specificamente per supportare la skill di ingestione PDF nella LLM Wiki della tesi. Il frontend è un layer di comodità: l'integrazione core usa direttamente l'endpoint locale esposto.

**Privacy:** tutto il processing avviene offline. Nessun dato viene inviato a servizi cloud.

### Requisiti

Il tool richiede due sole dipendenze: **Ollama** con il modello `glm-ocr:latest` (`ollama pull glm-ocr:latest`) e **Poppler** per la conversione PDF → immagini (`brew install poppler` su macOS, `apt-get install poppler-utils` su Linux).

Il frontend Streamlit incluso nella repository è un layer di comodità per uso interattivo — non è necessario per l'integrazione core, che chiama direttamente l'endpoint Ollama locale (`http://localhost:11434`).

### Installazione e avvio (frontend opzionale)

```bash
git clone https://github.com/ghMellow/local-glm-ocr-pdf-to-markdown
cd local-glm-ocr-pdf-to-markdown

poetry install
poetry run streamlit run app.py
```

L'interfaccia si apre nel browser con output in streaming in tempo reale.

### Integrazione con la LLM Wiki

L'endpoint esposto localmente da Ollama (`http://localhost:11434`) è quello che la skill di ingestione nella wiki chiama direttamente. Il flusso è:

1. L'agente riceve un PDF da processare
2. Segue le istruzioni della skill `SKILL-thesis-ingest.md`
3. Chiama l'endpoint locale, passa il PDF
4. Riceve il Markdown estratto
5. Lo sposta nella cartella corretta e aggiorna la wiki

---

## Configurazione globale — `config.py` (Esperimento MAS)

```python
MODELS = {
    "expert_1A":    "qwen3:9b",
    "beginner_1A":  "qwen3:9b",
    "expert_1B":    "qwen3:9b",
    "beginner_1B":  "deepseek-r1:7b",
    "judge":        "qwen3:9b",
}

OLLAMA_BASE_URL = "http://localhost:11434"

TEMPERATURE     = 0.0
MAX_RETRIES     = 3
REPETITIONS     = 3
TASKS_PATH      = "docs/tasks/"
RESULTS_PATH    = "results/"
```

Per modificare i modelli o i parametri dell'esperimento, è sufficiente aggiornare questo file senza toccare il codice degli agenti.

---

## Roadmap

| Fase | Stato | Descrizione |
|---|---|---|
| Wiki + ingestione PDF | ✅ Completato | LLM Wiki + skill PDF→MD |
| PDF to Markdown tool | ✅ Completato | GLM-OCR locale con interfaccia web |
| Esperimento MAS Memory | ✅ Completato | Benchmark multi-agente su task 5G |
| Framework CDT core | 🔄 In sviluppo | Simulatore 5G + Eclipse Ditto + Neo4j + LangGraph |

---

## Note per i Relatori

Il codice dell'esperimento MAS Memory è progettato per essere eseguibile su hardware consumer con GPU dedicata o CPU sufficientemente potente. I modelli Qwen3 9B e DeepSeek-R1 7B richiedono rispettivamente circa 6-8 GB di VRAM (o RAM unificata su Apple Silicon).

Le repository sono indipendenti e possono essere clonate e usate separatamente. L'unica dipendenza trasversale è Ollama, che deve essere attivo localmente per qualsiasi componente che usa LLM.

Per qualsiasi problema di replicazione, il file `docs/setup.md` nell'esperimento MAS Memory contiene la documentazione tecnica dettagliata con lo schema dell'architettura LangGraph, i system prompt degli agenti e le istruzioni complete di configurazione.
