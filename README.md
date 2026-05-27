# Quadre Language

> *Un linguaggio dichiarativo e reattivo per WebGames e WebApp*

**Quadre** è una DSL (Domain Specific Language) pensata per sviluppare WebGames e WebApp event-driven senza la complessità di HTML + JavaScript.

```xml
<!-- Un semplice esempio Quadre -->
<entity onstage>
    <var punteggio: int = 0></var>
</entity>

<when event=periferiche.mouse.click; target=HUD.btn>
    [set onstage.punteggio = (onstage.punteggio + 1)]
</when>
```

---

## Il principio fondamentale — Solido/Liquido

| Stato | Sintassi | Funzione |
|---|---|---|
| **Solido** | `< >` | Struttura persistente |
| **Liquido** | `[ ]` | Azione runtime |

Tutto ciò che **esiste** è solido. Tutto ciò che **accade** è liquido.

---

## Struttura del repository

```
quadre-lang/
├── README.md
├── spec/
│   └── quadre-reference-v0.1.md   ← Reference completa del linguaggio
├── runtime/
│   └── quadre-runtime.js           ← Interprete JavaScript
├── examples/
│   └── pacman/
│       ├── pacman.qde              ← Codice sorgente Quadre
│       └── index.html              ← Demo giocabile nel browser
└── docs/
    └── philosophy.md               ← Filosofia e motivazioni
```

---

## Demo

Apri `examples/pacman/index.html` nel browser per vedere Quadre in azione.

---

## Roadmap

| Versione | Contenuto |
|---|---|
| **v0.1** | Core language — eventi, logica, UI testuale, game loop |
| **v0.2** | Grafica — sprite, canvas, animazioni |
| **v0.3** | Audio — suoni, musica |
| **v0.4** | Rete — multiplayer, sync |
| **v1.0** | Runtime completo, playground online |

---

## File `.qde`

I programmi Quadre hanno estensione `.qde` — da *Quadre Declarative Engine*.

---

*Quadre Language v0.1 — Licenza MIT*
