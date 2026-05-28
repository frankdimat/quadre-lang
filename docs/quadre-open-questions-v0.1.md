# Quadre Language — Open Questions & Known Limitations v0.1

> Questo documento raccoglie le questioni aperte, le limitazioni note e le decisioni di design
> ancora da prendere nel linguaggio Quadre v0.1. Ogni punto è un candidato per le versioni future.

---

## 1. Matrici: creazione e binding reattivo

### 1.1 Creazione con parametri *(v0.1)*

```xml
<var griglia: matrix = matrix(10; 10; 0)></var>
```

Parametri: righe, colonne, valore iniziale di ogni cella.

### 1.2 Binding reattivo per celle *(v0.2 — da definire)*

Quando si modifica una singola cella con `[set m[r][c] = valore]`, il rendering 
dell'interfaccia aggiorna automaticamente **solo quella cella** — non l'intera matrice. 
Questo è essenziale per performance su griglie grandi con game loop a 60 tick/s.

Il runtime distingue automaticamente due casi:

| Operazione | Aggiornamento UI |
|---|---|
| `[set m[r][c] = valore]` | incrementale — solo la cella modificata |
| `[reset m]` | totale — tutta la matrice |
| `[set m = clone(...)]` | totale — tutta la matrice |

**Decisione proposta:** il runtime traccia internamente quale operazione ha generato 
il cambiamento e ottimizza il ridisegno di conseguenza. Il programmatore non deve fare 
nulla di esplicito — il comportamento corretto è automatico.

---

## 2. Ricorsione

**Problema:** La ricorsione diretta o indiretta tra routine non è definita. 
Nel modello attuale i job sono accodati sequenzialmente — una ricorsione 
infinita riempirebbe la coda senza controllo.

**Decisione v0.1:** Ricorsione **non supportata**.

**Caso d'uso reale — flood fill nel Campo Minato:**

Il caso più naturale che richiede ricorsione è la rivelazione a cascata 
delle celle vuote adiacenti. Senza ricorsione questo algoritmo non è 
esprimibile in modo pulito:

```xml
<routine rivela_cella max_depth=64>
    [var r = input.r]
    [var c = input.c]

    <!-- Se fuori limiti o già rivelata, esci -->
    [test condition=(r < 0 OR r >= RIGHE OR c < 0 OR c >= COLONNE);
        <then>[return]</then>]
    [test condition=(campo.visibile[r][c] != NASCOSTA);
        <then>[return]</then>]

    <!-- Rivela la cella -->
    [set campo.visibile[r][c] = VUOTA_RIVELATA]
    [set partita.celle_rivelate = (partita.celle_rivelate + 1)]

    <!-- Se vuota, rivela ricorsivamente le adiacenti -->
    [test condition=(campo.valori[r][c] == 0);
        <then>
            [routine rivela_cella; r=(r-1); c=(c-1)]
            [routine rivela_cella; r=(r-1); c=c]
            [routine rivela_cella; r=(r-1); c=(c+1)]
            [routine rivela_cella; r=r;     c=(c-1)]
            [routine rivela_cella; r=r;     c=(c+1)]
            [routine rivela_cella; r=(r+1); c=(c-1)]
            [routine rivela_cella; r=(r+1); c=c]
            [routine rivela_cella; r=(r+1); c=(c+1)]
        </then>]
</routine>
```

`max_depth=64` su una griglia 8x8 è sufficiente — il flood fill 
non può scendere più in profondità del numero totale di celle.

**Proposta per v0.2:** Supporto alla ricorsione con limite esplicito 
`max_depth` dichiarato nel tag della routine. Il runtime genera un 
errore esplicito se il limite viene superato.

**Stato:** Da implementare in v0.2. Il campo minato è l'esempio 
di riferimento per questa feature.

---

## 3. Accesso a coordinate su eventi mouse in griglia

**Problema:** Quando un `<when event=periferiche.mouse.click>` ha come target un elemento con `render="matrix"`, manca un modo per sapere su quale cella è avvenuto il click.

**Proposta per v0.2:**

Aggiungere proprietà native all'oggetto `event` quando il target è una griglia:

| Proprietà | Tipo | Descrizione |
|---|---|---|
| `event.row` | int | riga della cella cliccata |
| `event.col` | int | colonna della cella cliccata |
| `event.x` | int | coordinata pixel X assoluta |
| `event.y` | int | coordinata pixel Y assoluta |

**Esempio d'uso:**
```xml
<when event=periferiche.mouse.click; target=HUD.griglia>
    [test condition=(mappa.griglia[event.col][event.row] == VUOTO);
        <then>
            [set mappa.griglia[event.col][event.row] = BANDIERA]
        </then>]
</when>
```

**Stato:** Da aggiungere in v0.2 insieme al sistema grafico.

---

## 4. Renderer personalizzati

**Problema:** `render="matrix"` è l'unico renderer predefinito. Per giochi con grafica personalizzata (es. campo minato con icone diverse, mappe con terreni multipli) serve un meccanismo per definire renderer custom.

**Due approcci possibili:**

**Approccio A — renderer esterno (linguaggio ospite)**
Il programmatore registra una funzione JavaScript esterna:
```javascript
QuadreRuntime.registerRenderer('campo_minato', (value, ctx) => { ... });
```
Pro: massima flessibilità. Contro: rompe l'incapsulamento, richiede JavaScript.

**Approccio B — renderer dichiarativo in Quadre** *(preferito)*
```xml
<renderer campo_minato>
    <case value=0> <sprite src="vuoto.png"></sprite> </case>
    <case value=1> <sprite src="mina.png"></sprite> </case>
    <case value=9> <sprite src="bandiera.png"></sprite> </case>
    <case value="default"> <sprite src="coperta.png"></sprite> </case>
</renderer>
```

Usato poi in:
```xml
<text griglia bind=mappa.griglia render="campo_minato"></text>
```

**Stato:** Approccio B da progettare e implementare in v0.2 insieme al sistema grafico.

---

## 5. Tipi mancanti

I seguenti tipi potrebbero essere necessari per applicazioni reali:

| Tipo | Descrizione | Versione prevista |
|---|---|---|
| `float` | numero decimale | v0.2 |
| `color` | colore esadecimale o RGB | v0.2 |
| `sprite` | riferimento a risorsa grafica | v0.2 |
| `sound` | riferimento a risorsa audio | v0.3 |
| `map` | dizionario chiave-valore | v0.3 |

---

## 6. Gestione degli errori

**Problema:** Non esiste un meccanismo per gestire errori a runtime (tipo incompatibile, indice fuori range, divisione per zero).

**Proposta per v0.2:**
```xml
[try
    <do>
        [set risultato = (valore / divisore)]
    </do>
    <catch error="divisione_per_zero">
        [set risultato = 0]
        [expose sistema.errore_calcolo]
    </catch>]
```

**Stato:** Da progettare per v0.2.

---

*Quadre Language v0.1 — Open Questions*
*Documento aggiornato con il progredire del linguaggio.*
