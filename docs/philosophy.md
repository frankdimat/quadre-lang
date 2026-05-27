# Quadre — Filosofia e Motivazioni

## Il problema

HTML + JavaScript sono strumenti potenti, ma la loro complessità è sproporzionata per chi vuole costruire un WebGame semplice o una webapp event-driven.

Per fare un gioco minimale servono:
- HTML per la struttura
- CSS per il layout
- JavaScript per la logica
- Un framework (React, Vue) per la reattività
- Un game loop manuale
- Gestione manuale degli eventi

Questo stack richiede anni di esperienza per essere padroneggiato. E la maggior parte della complessità non serve per un gioco di carte, un boardgame online, o una semplice webapp interattiva.

## La soluzione

Quadre separa il problema in due livelli chiari:

**Cosa esiste** — dichiarato una volta, in modo statico, con la sintassi `< >`.

**Cosa accade** — espresso come impulsi nel tempo, con la sintassi `[ ]`.

Questa distinzione, che chiamiamo **dualismo Solido/Liquido**, è il cuore del linguaggio.

## Perché dichiarativo?

I linguaggi imperativi (JavaScript, Python, C) ti dicono *come* fare le cose. I linguaggi dichiarativi ti permettono di dire *cosa* vuoi ottenere.

In Quadre non scrivi:

```javascript
document.querySelector('#btn').addEventListener('click', function() {
    state.score += 100;
    document.querySelector('#score').textContent = state.score;
});
```

Scrivi:

```xml
<when event=periferiche.mouse.click; target=HUD.btn>
    [set onstage.punteggio = (onstage.punteggio + 100)]
</when>
```

Il binding reattivo aggiorna l'UI automaticamente. Non devi pensare al DOM.

## Perché non usare GameMaker o Godot?

GameMaker e Godot sono eccellenti per giochi desktop e mobile. Ma richiedono un IDE pesante, una curva di apprendimento significativa, e non sono pensati per il web.

Quadre è web-first — un file `.qde` che gira nel browser, senza installare nulla.

## La nicchia

Quadre è pensato per:

- **Educazione** — insegnare i concetti di programmazione event-driven attraverso i giochi
- **Boardgame online** — prototipare e pubblicare giochi da tavolo sul web
- **Prototipazione rapida** — testare meccaniche di gioco senza scrivere codice vero
- **WebApp semplici** — form, dashboard, strumenti interattivi

## I principi di design

1. **Leggibilità prima di tutto** — il codice deve essere comprensibile da chi non è programmatore
2. **Un concetto, uno strumento** — ogni costrutto ha una responsabilità precisa e non si sovrappone agli altri
3. **Errori espliciti** — nessuna coercion implicita, nessun comportamento silenzioso
4. **Web-first** — nessuna dipendenza esterna, gira nel browser

---

*Quadre Language v0.1*
