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
## 2. Ciclo `[while]`

**Problema:** Non esiste un costrutto per loop condizionali. `[cycle]` copre il caso `for` (range numerico o collezione), ma non il caso `while` (condizione booleana).

**Proposta per v0.2:**
```xml
[while condition=(x > 0);
    <loop>
        [set x = (x - 1)]
    </loop>]
```

**Workaround attuale:** Usare `[watch]` per reagire ai cambiamenti di variabile, o `[cycle]` con un range sufficientemente grande.

**Stato:** Lacuna confermata — da aggiungere in v0.2.

---

## 3. Ricorsione

**Problema:** La ricorsione diretta o indiretta tra routine non è definita. Nel modello attuale i job sono accodati sequenzialmente — una ricorsione infinita riempirebbe la coda senza controllo.

**Decisione v0.1:** Ricorsione **non supportata**.

**Proposta per versioni future:**
```xml
<routine fattoriale max_depth=100>
    [test condition=(input.n <= 1);
        <then>
            <return 1></return>
        </then>
        <else>
            [var sub = routine(fattoriale; n=(input.n - 1))]
            <return (input.n * sub)></return>
        </else>]
</routine>
```

Con `max_depth` come limite esplicito di profondità — genera errore se superato.

**Stato:** Da valutare per v0.3+.

---

## 4. Accesso a coordinate su eventi mouse in griglia

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

## 5. Renderer personalizzati

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

## 6. `<return>` e `[return]`

**Problema:** Il linguaggio v0.1 non ha un meccanismo di ritorno da routine o di uscita anticipata da job.

**Proposta approvata per v0.1:**

`<return>` — valore di ritorno da routine (tag solido):
```xml
<routine somma>
    [var risultato = (input.a + input.b)]
    <return risultato></return>
</routine>

[var totale = routine(somma; a=5; b=3)]
```

`[return]` — uscita anticipata da job (verbo liquido):
```xml
<job processa_dati>
    [test condition=(errore == true);
        <then>
            [return]
        </then>]
    <!-- continua solo se nessun errore -->
</job>
```

**Nota semantica importante:** `[return]` dentro un `[cycle]` esce dall'intero job, non solo dal ciclo corrente.

`[return condition=...; value=...]` — return condizionale *(roadmap v0.2)*:
```xml
[return condition=(x > 10); value=x]
```

**Stato:** Da aggiungere alla reference v0.1 e all'Appendice A.

---

## 7. Tipi mancanti

I seguenti tipi potrebbero essere necessari per applicazioni reali:

| Tipo | Descrizione | Versione prevista |
|---|---|---|
| `float` | numero decimale | v0.2 |
| `color` | colore esadecimale o RGB | v0.2 |
| `sprite` | riferimento a risorsa grafica | v0.2 |
| `sound` | riferimento a risorsa audio | v0.3 |
| `map` | dizionario chiave-valore | v0.3 |

---

## 8. Gestione degli errori

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
