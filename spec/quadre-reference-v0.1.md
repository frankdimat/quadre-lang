# Quadre Language — Reference Guide v0.1

> *Un linguaggio dichiarativo e reattivo per WebGames e WebApp*
> *creato e ideato da Francesco Di Matteo (frankdimat@gmail.com)*

---

## Prefazione

### Filosofia del linguaggio

Quadre è una DSL (Domain Specific Language) reattiva e topologica progettata per:

- orchestrazione di interfacce utente
- sviluppo di WebGames
- logica di boardgame
- sistemi event-driven

Quadre nasce da una convinzione semplice: la complessità di HTML + JavaScript è sproporzionata rispetto ai problemi che la maggior parte degli sviluppatori di giochi e webapp deve risolvere. Quadre abbassa il costo di ingresso senza rinunciare alla potenza espressiva.

### Il dualismo Solido/Liquido

Il principio fondamentale di Quadre è la separazione netta tra due stati del programma:

| Stato | Sintassi | Funzione |
|---|---|---|
| **Solido** | `< >` | Struttura persistente, dichiarata all'avvio |
| **Liquido** | `[ ]` | Azione runtime, impulso nel tempo |

Tutto ciò che **esiste** nel mondo del programma è solido.
Tutto ciò che **accade** nel tempo è liquido.

Questa distinzione non è stilistica — è ontologica. Il compilatore la rispetta rigorosamente.

### Come leggere questa reference

Ogni costrutto è documentato con:
- **Categoria** — Solido o Liquido
- **Sintassi** — forma canonica con parametri
- **Comportamento** — semantica precisa
- **Esempio** — codice reale
- **Note** — casi limite e relazioni con altri costrutti

I file Quadre hanno estensione `.qde`.

---

## Parte 1 — Fondamenti

### 1. Struttura di un programma `.qde`

Un programma Quadre è composto da blocchi di primo livello nell'ordine seguente (consigliato):

```xml
<!-- 1. Costanti globali -->
<const NOME = valore></const>

<!-- 2. Entità globali -->
<entity nome> ... </entity>

<!-- 3. Template oggetti -->
<object Nome> ... </object>

<!-- 4. Istanze -->
<NomeOggetto istanza; parametri></NomeOggetto>

<!-- 5. Sezioni (UI, logica, eventi) -->
<section nome> ... </section>
```

Non esiste un punto di ingresso esplicito (nessun `main`). Il programma reagisce agli eventi — il primo evento è l'avvio del runtime (`runtime.start`).

---

### 2. Sezioni e scope

**Categoria:** Solido

```xml
<section nome_sezione>
    ...
</section>
```

Le sezioni sono namespace geografici e contenitori logici. Organizzano il programma per responsabilità.

**Regole di scope:**

| Scope | Posizione | Come si accede |
|---|---|---|
| Locale | dentro la stessa section | nome diretto |
| Assoluto | section diversa | `sezione.elemento` |
| Globale | entity, object, const | nome diretto ovunque |

**Esempio:**
```xml
<section logica>
    <job controlla>
        [job altro_job]                        <!-- scope locale -->
        [job eventi.gestisci_input]            <!-- scope assoluto -->
    </job>
</section>
```

**Note:**
- Le sezioni non possono essere annidate
- I nomi di sezione devono essere unici nel programma

---

### 3. Costanti `<const>`

**Categoria:** Solido

```xml
<const NOME = valore></const>
```

Dichiara un valore immutabile riutilizzabile in tutto il programma. Per convenzione i nomi sono in MAIUSCOLO.

**Esempio:**
```xml
<const VELOCE = 1></const>
<const NORMALE = 6></const>
<const LENTO = 12></const>

<const VUOTO = 0></const>
<const MURO = 1></const>
<const PALLINO = 2></const>
```

**Note:**
- Le costanti non possono essere modificate a runtime
- Sono sostituite dal parser prima dell'esecuzione — nessun overhead
- Rendono il codice autodocumentante

---

### 4. Tipi di dato

Quadre supporta i seguenti tipi nativi:

| Tipo | Descrizione | Esempio |
|---|---|---|
| `int` | Numero intero | `42`, `-10`, `0` |
| `string` | Stringa di testo | `"in_corso"`, `"vittoria"` |
| `bool` | Booleano | `true`, `false` |
| `list` | Lista ordinata di valori | `[1, 2, 3]`, `["orco", "goblin"]` |
| `matrix` | Matrice bidimensionale | `[[1,0],[0,1]]` |
| `range` | Intervallo numerico | `(1..10)` |

**Liste:**
```xml
<var nemici: list = ["orco", "goblin", "troll"]></var>
```
Accesso per indice:
```xml
[set primo = nemici[0]]
```

**Matrix:**
```xml
<var griglia: matrix = [
    [1,1,1,1,1],
    [1,0,0,0,1],
    [1,1,1,1,1]
]></var>
```
Accesso per coordinate:
```xml
[test condition=(griglia[x][y] == MURO)]
```

**Note:**
- Nessuna coercion implicita tra tipi diversi
- Un confronto tra tipi incompatibili genera errore esplicito
- `range` è usato principalmente come sorgente per `[cycle]`

---

## Parte 2 — Stato e Variabili

### 5. Variabili statiche `<var>`

**Categoria:** Solido

```xml
<var nome: tipo = valore></var>
```

Dichiara una variabile persistente con tipo e valore iniziale opzionale.

**Esempio:**
```xml
<var vita: int = 100></var>
<var stato: string = "in_corso"></var>
<var potenziato: bool = false></var>
```

**Note:**
- Una `<var>` senza valore iniziale è `null` fino al primo `[set]`
- Il valore iniziale è il riferimento per `[reset]`
- `<var>` e `[set]` sono speculari: uno dichiara, l'altro modifica

---

### 6. Valori derivati `<computed>`

**Categoria:** Solido

```xml
<computed nome = espressione></computed>
```

Dichiara un valore che si aggiorna automaticamente ogni volta che le variabili da cui dipende cambiano.

**Esempio:**
```xml
<computed pallini_rimasti = count(griglia == PALLINO)></computed>
<computed giocatore_vivo = (giocatore.vite > 0)></computed>
```

**Differenza con `<var>`:**

| | `<var>` | `<computed>` |
|---|---|---|
| Si aggiorna con | `[set]` esplicito | automaticamente |
| Modificabile | sì | no |
| Uso | valori imperativi | valori derivati |

---

### 7. Assegnazione `[set]`

**Categoria:** Liquido

```xml
[set TARGET = VALORE]
```

**Forme:**
```xml
[set giocatore.vita = 100]
[set giocatore.vita = (giocatore.vita - 10)]
[set griglia[x][y] = VUOTO]
[set giocatore {
    vita = 100
    punteggio = 0
    potenziato = false
}]
```

---

### 8. Reset `[reset]` e modalità checkpoint

**Categoria:** Liquido

```xml
[reset TARGET]
[reset TARGET; mode=checkpoint]
[checkpoint TARGET]
```

**Modalità:**

| Modalità | Riferimento |
|---|---|
| default | valore dichiarato in `<var>` |
| `checkpoint` | ultimo snapshot salvato con `[checkpoint]` |

**Esempio:**
```xml
[checkpoint giocatore]
<!-- ... danni ... -->
[reset giocatore; mode=checkpoint]
[reset giocatore]          <!-- torna ai valori iniziali -->
[reset onstage]            <!-- resetta tutta l'entity -->
```

---

### 9. Snapshot `[snapshot]` e `[restore]`

**Categoria:** Liquido

```xml
[snapshot NOME]
[snapshot NOME; scope=TARGET]
[restore NOME]
```

Copia serializzabile dello stato per salvataggi e replay.

| | `[checkpoint]` | `[snapshot]` |
|---|---|---|
| Persistenza | runtime | persistente |
| Nominato | no | sì |
| Uso | reset mid-game | save, replay |

---

### 10. Clone `clone()`

```xml
[set display.schermo = clone(mappa.griglia)]
```

Restituisce una copia indipendente di una struttura. Modificare la copia non altera l'originale.

---

## Parte 3 — Oggetti ed Entità

### 11. Template oggetto `<object>`

**Categoria:** Solido

```xml
<object NomeOggetto>
    <var proprietà: tipo = valore></var>
    <job nome> ... </job>
    <routine nome> ... </routine>
</object>
```

Definisce il template di un elemento fisico del mondo. Le istanze si creano separatamente.

---

### 12. Entità astratte `<entity>`

**Categoria:** Solido

```xml
<entity nome>
    <var proprietà: tipo = valore></var>
    <computed nome = espressione></computed>
</entity>
```

Contenitore di stato astratto — non ha posizione, forma o rappresentazione visiva.

| | `<object>` | `<entity>` |
|---|---|---|
| Natura | fisico | astratto |
| Istanze multiple | sì | no (singleton) |
| Visibile | sì | no |

---

### 13. Istanze

**Categoria:** Solido

```xml
<NomeOggetto nome_istanza;
    proprietà1=valore1;
    proprietà2=valore2>
</NomeOggetto>
```

**Esempio:**
```xml
<Fantasma blinky;
    x=4; y=4;
    velocita=NORMALE;
    livello_attivo=1;
    errore=2>
</Fantasma>
```

---

### 14. `self` e `instances`

**`self`** — riferimento all'istanza corrente dentro un `<object>`:
```xml
[set self.vita = (self.vita - 10)]
```

**`instances`** — collezione di tutte le istanze attive:
```xml
[cycle source=Fantasma.instances; item=f; ...]
```

---

### 15. Entità di sistema predefinite

```xml
<entity runtime>
    <var tick_rate: int = 60></var>
</entity>
```

---

## Parte 4 — Interfaccia Utente

### 16. `<container>`

```xml
<container nome [color="#hex"] [width=N] [height=N]>
    ...
</container>
```

Nodo visuale contenitore. Supporta annidamento.

---

### 17. `<text>` — `value` e `bind`

```xml
<text nome value="testo statico"></text>
<text nome bind=percorso.variabile></text>
```

`value` e `bind` sono mutualmente esclusivi. `bind` è reattivo — si aggiorna automaticamente.

---

### 18. `render="matrix"` e `overlay`

```xml
<text schermo
    bind=mappa.griglia
    render="matrix"
    overlay=Pacman.instances; Fantasma.instances>
</text>
```

Il renderer sovrappone le istanze alla griglia senza modificare la struttura originale.

---

### 19. `[transition]` — animazioni dichiarative

```xml
[transition TARGET.proprietà;
    from=VALORE; to=VALORE;
    duration=MS;
    easing=ease_in_out]
```

Easing disponibili: `linear`, `ease_in`, `ease_out`, `ease_in_out`, `bounce`.

---

## Parte 5 — Blocchi Esecutivi

### 20. Job `<job>` — il blocco risoluto

Il job è un blocco risoluto — conosce già il suo scopo e agisce direttamente sul mondo, senza bisogno di parametri esterni.

```xml
<job nome>
    [istruzione]
    ...
</job>
```

Chiamata:
```xml
[job nome]
[job sezione.nome]
```

---

### 21. Routine `<routine>` — il blocco specializzato

La routine è un blocco specializzato — ha bisogno di conoscere il contesto prima di agire. Riceve i dati via oggetto `input`.

```xml
<routine nome>
    [istruzione usando input.parametro]
</routine>
```

Chiamata:
```xml
[routine nome; param1=valore; param2=valore]
[routine nome {
    param1 = valore
    param2 = valore
}]
```

---

### 22. Group `<group>` — raggruppamento tecnico

Sequenza nominata senza parametri. Esiste per evitare ripetizioni tecniche — non rappresenta un'azione del mondo.

```xml
<group nome>
    [istruzione]
</group>
```

---

### 23. Variabili locali `[var]`

```xml
[var nome: tipo = valore]
```
### 23bis. Return — `<return>` e `[return]`

Quadre distingue due forme di ritorno, entrambe runtime, ma con semantica differente:

| Forma | Tipo | Scopo |
|---|---|---|
| `<return>` | nodo runtime semantico | restituzione di un valore |
| `[return]` | istruzione runtime di controllo | uscita immediata dal flusso |

Le due istruzioni non sono intercambiabili.

---

### `<return>` — restituzione di valore da routine

```quadre
<routine somma>
    [var risultato = (input.a + input.b)]
    <return risultato></return>
</routine>

[var totale = routine(somma; a=5; b=3)]
```

Una routine può avere più `<return>` — il primo eseguito termina immediatamente.

```quadre
<routine massimo>
    [test condition=(input.a > input.b);
        <then><return input.a></return></then>]
    <return input.b></return>
</routine>
```

Se la routine termina senza `<return>`, il valore restituito è `null`.

---

### `[return]` — uscita anticipata

Interrompe immediatamente il job, la routine o il blocco corrente senza restituire alcun valore. Usato come guardia di errore o early exit.

```quadre
<job processa_dati>
    [test condition=(errore == true);
        <then>[return]</then>]
    [job continua_elaborazione]
</job>
```

**Nota:** dentro un `[cycle]`, `[return]` termina l'intero job — non solo il ciclo.
Per interrompere solo il ciclo si usa `[break]` *(roadmap v0.2)*.

---

### Differenza concettuale

| | `<return>` | `[return]` |
|---|---|---|
| Restituisce un valore | sì | no |
| Termina il flusso | sì | sì |
| Usato in | routine | job / routine / cycle |
| Equivalente classico | `return value` | `return;` |

---

### Nota sul dualismo — tag statici e tag runtime

In Quadre i tag `< >` possono essere sia statici che runtime.
La distinzione non è tra compile-time e runtime, ma tra:

| Sintassi | Ruolo |
|---|---|
| `[ ]` | azione, effetto, controllo |
| `< >` | nodo semantico o strutturale |

Esempi: `[set]` modifica lo stato — `[return]` interrompe il flusso —
`<return>` produce un valore — `<then>` e `<else>` organizzano i rami —
`<loop>` definisce il corpo logico di un'iterazione.

**`[return condition=...; value=...]` — return condizionale** *(roadmap v0.2)*

```xml
[return condition=(x > 10); value=x]
```
Esiste solo per la durata del blocco corrente. Corrispettivo liquido di `<var>`.

---

## Parte 6 — Controllo di Flusso

### 24. Condizione effimera `[test]`

```xml
[test condition=(ESPRESSIONE);
    <then>
        [istruzione]
    </then>
    <else>
        [istruzione]
    </else>
]
```

`<else>` è opzionale. Effimero — non persistente.

---

### 25. Condizione persistente `[evaluate]`

```xml
[evaluate NOME condition=(ESPRESSIONE)]
```

Condizione nominata e riutilizzabile. Restituisce un booleano — non esegue rami.

```xml
[evaluate livello_completato condition=(onstage.pallini_rimasti == 0)]

<when event=...; active=evaluate livello_completato>
[test condition=(evaluate livello_completato); ...]
```

---

### 26. Selezione `[match]` / `<case>`

```xml
[match target=VARIABILE;
    <case value=VALORE1>
        [istruzione]
    </case>
    <case value="default">
        [istruzione]
    </case>
]
```

---

### 27. Iterazione `[cycle]` / `<loop>`

```xml
[cycle source=COLLEZIONE; item=VAR;
    <loop>
        [istruzione]
    </loop>
]

[cycle range=(N..M); item=VAR;
    <loop>
        [istruzione]
    </loop>
]
```

---

### 28. Stato di macchina `[state]`

```xml
[state NOME]
```

Un solo stato attivo alla volta. Usabile come guardia:

```xml
<when event=...; active=state(battaglia)>
```

---

## Parte 7 — Sistema Reattivo

### 29. Osservatore eventi `<when>`

```xml
<when event=NAMESPACE.TIPO [; target=ELEMENTO] [; active=CONDIZIONE]>
    [istruzione]
</when>
```

`active` può essere:
```xml
active=condition(espressione)
active=evaluate nome
active=state(nome)
```

---

### 30. Osservatore variabile `[watch]`

```xml
[watch VARIABILE]
    [istruzione]
[/watch]
```

Si attiva solo al cambiamento del valore — non ad ogni tick.

---

### 31. Segnale locale `[signal]`

```xml
[signal NOME]
```

Evento locale intercettabile solo nella stessa sezione.

---

### 32. Evento globale `[expose]`

```xml
[expose NAMESPACE.NOME]
```

Evento globale intercettabile da qualsiasi `<when>`.

Wildcard:
```xml
<when event=sistema.*>
    [match target=event.type; ...]
</when>
```

---

### 33. Scheduling `[delay]`

```xml
[delay MS; <then> [istruzione] </then>]
[delay NOME; MS; repeat=true; <then> [istruzione] </then>]
[cancel NOME]
```

---

## Parte 8 — Namespace degli Eventi

### 34. `periferiche.*`

```
periferiche.keyboard.arrow
periferiche.keyboard.key_press
periferiche.keyboard.escape
periferiche.keyboard.space
periferiche.keyboard.enter
periferiche.mouse.click
periferiche.mouse.double_click
periferiche.mouse.move
periferiche.mouse.over
periferiche.mouse.out
periferiche.gamepad.button_a
periferiche.gamepad.button_b
periferiche.gamepad.stick_left
periferiche.gamepad.stick_right
```

---

### 35. `runtime.*`

```
runtime.tick          <!-- ogni frame -->
runtime.start         <!-- avvio programma -->
runtime.collision     <!-- collisione tra oggetti -->
runtime.proximity     <!-- oggetti entro una distanza -->
```

```xml
<when event=runtime.collision; source=Pacman; target=Pallino>
<when event=runtime.proximity; source=Pacman; target=Fantasma; range=2>
```

---

### 36. `sistema.*`

Eventi custom definiti dallo sviluppatore con `[expose]`.

---

### 37. Oggetto `event`

| Proprietà | Tipo | Descrizione |
|---|---|---|
| `event.type` | string | nome completo dell'evento |
| `event.delta_x` | int | spostamento X (keyboard.arrow) |
| `event.delta_y` | int | spostamento Y (keyboard.arrow) |
| `event.source` | object | oggetto sorgente |
| `event.target` | object | oggetto destinazione |
| `event.timestamp` | int | tick al momento dell'evento |

---

## Parte 9 — Operatori ed Espressioni

### 38. Operatori

| Operatore | Tipo | Esempio |
|---|---|---|
| `+` | aritmetico | `(a + b)` |
| `-` | aritmetico | `(a - b)` |
| `*` | aritmetico | `(a * b)` |
| `/` | aritmetico | `(a / b)` |
| `mod` | aritmetico | `(tick mod velocita)` |
| `==` | confronto | `(stato == "in_corso")` |
| `!=` | confronto | `(vita != 0)` |
| `>` | confronto | `(vita > 0)` |
| `<` | confronto | `(vita < 100)` |
| `>=` | confronto | `(livello >= 2)` |
| `<=` | confronto | `(vita <= 0)` |
| `AND` | logico | `(x == 1 AND y == 1)` |
| `OR` | logico | `(a OR b)` |
| `NOT` | logico | `(NOT potenziato)` |

---

## Parte 10 — Funzioni Native

### 39. `count()`

```xml
count(mappa.griglia == PALLINO)
count(Fantasma.attivo == true)
count(nemici)
```

### 40. `clone()`

```xml
[set copia = clone(struttura)]
```

### 41. `random()`

```xml
[random dado; range=(1..6)]
[random delta; range=(-1..1)]
```

### 42. `[cache]`

```xml
[cache NOME;
    source=(ESPRESSIONE);
    ttl=MS]
```

---

## Parte 11 — Ottimizzazione e Concorrenza

### 43. `[pipeline]`

```xml
[pipeline]
    [set x = 1]
    [set y = 2]
    [routine aggiorna]
[/pipeline]
```

Operazioni atomiche e ottimizzate.

---

### 44. `[lock]`

```xml
[lock TARGET]
    [istruzione]
[/lock]
```

Esclusione mutua. Rilascio automatico al termine del blocco.

---

### 45. `[sync]` *(roadmap v0.4)*

```xml
[sync TARGET; mode=server_authoritative]
[sync TARGET; mode=client_to_server]
[sync TARGET; mode=broadcast]
```

---

## Appendici

### A. Tabella completa dei costrutti

| Costrutto | Categoria | Versione |
|---|---|---|
| `<const>` | Solido | v0.1 |
| `<var>` | Solido | v0.1 |
| `<computed>` | Solido | v0.1 |
| `<entity>` | Solido | v0.1 |
| `<object>` | Solido | v0.1 |
| `<section>` | Solido | v0.1 |
| `<container>` | Solido UI | v0.1 |
| `<text>` | Solido UI | v0.1 |
| `<job>` | Solido | v0.1 |
| `<routine>` | Solido | v0.1 |
| `<group>` | Solido | v0.1 |
| `<when>` | Solido | v0.1 |
| `[set]` | Liquido | v0.1 |
| `[reset]` | Liquido | v0.1 |
| `[checkpoint]` | Liquido | v0.1 |
| `[snapshot]` / `[restore]` | Liquido | v0.2 |
| `[var]` | Liquido | v0.1 |
| `[job]` | Liquido | v0.1 |
| `[routine]` | Liquido | v0.1 |
| `[group]` | Liquido | v0.1 |
| `[test]` | Liquido | v0.1 |
| `[evaluate]` | Liquido persistente | v0.1 |
| `[match]` / `<case>` | Liquido | v0.1 |
| `[cycle]` / `<loop>` | Liquido | v0.1 |
| `[state]` | Liquido | v0.1 |
| `[watch]` | Liquido persistente | v0.1 |
| `[signal]` | Liquido | v0.1 |
| `[expose]` | Liquido | v0.1 |
| `[delay]` / `[cancel]` | Liquido | v0.1 |
| `[transition]` | Liquido UI | v0.2 |
| `[cache]` | Liquido | v0.2 |
| `[pipeline]` | Liquido | v0.2 |
| `[lock]` | Liquido | v0.3 |
| `[sync]` | Liquido | v0.4 |
| `random()` | Funzione | v0.1 |
| `clone()` | Funzione | v0.1 |
| `count()` | Funzione | v0.1 |

---

### B. Roadmap versioni

| Versione | Contenuto |
|---|---|
| **v0.1** | Core language — eventi, logica, UI testuale, game loop |
| **v0.2** | Grafica — sprite, canvas, transition, animazioni |
| **v0.3** | Audio — suoni, musica; lock e concorrenza |
| **v0.4** | Rete — multiplayer, sync, leaderboard |
| **v1.0** | Runtime completo, playground online, IDE web |

---

### C. Esempio completo — pacman.qde

*(vedi Appendice F nella versione estesa)*

---

*Quadre Language v0.1*
*Documentazione soggetta ad aggiornamenti con il progredire del linguaggio.*
