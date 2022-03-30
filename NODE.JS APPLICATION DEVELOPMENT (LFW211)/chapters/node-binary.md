# THE NODE BINARY

È possibile eseguire il parse di un'applicazione JavaScript, per fare il check della sintassi, utilizzando il comando

```bash
node --check app.js
# oppure
node -c app.js
```

Se nessun output è restituito allora il parse è stato fatto senza errori, altrimenti viene stampato l'errore.

Node può anche eseguire una valutazione del codice direttamente sulla shell. Si possono utilizzare le opzioni -p o --print che fanno una valutazione dell'espressione e stampano il risultato, oppure si può utilizzare -e o --eval che esegue solamente una valutazione.

```bash
$> node --print "1+1"
2
```

```bash
$> node --eval "1+1"
# non stampa nulla in quanto l'espressione è solamente valutata
```

```bash
$> node -e "console.log(1+1)"
2
# stampa 2 perché console.log è utilizzato per scrivere l'output
```

```bash
$> node -p "console.log(1+1)"
2
undefined
# stampa 2 e poi undefined in quanto prima viene stampato l'argomento di console.log, poi viene stampato il risultato di console.log che è undefined.
```

Possiamo accedere a tutti i moduli core di NodeJS direttamente utilizzando il loro namespace.

```bash
$> node -p "fs.readdirSync('.').filter((f) => /.js$/.test(f))"
# stampa tutti i file con estensione .js presenti nella directory corrente.
```

Grazie al fatto che NodeJS è multi piattaforma, possiamo eseguire il comando precedente in tutti i SO e ottenere sempre lo stesso risultato.

L'opzione -r o --require può essere utilizzata per caricare un modulo CommonJS prima di ogni altra cosa. Se ad esempio abbiamo il seguente file

```js
#preload.js
console.log('preload.js: this is preloaded')
```

e il seguente file

```js
#app.js
console.log('app.js: this is the main file')
```

il risultato del comando `node -r ./preload.js app.js` è 

```bash
$> node -r ./preload.js app.js
# preload.js: this is preloaded
# app.js: this is the main file
```

Lo stack trace è un "testo" che viene generato quando un qualsiasi errore (`Error`) viene sollevato dalla nostra applicazione. Di default lo stack trace ha dei limiti, ovvero contiene gli ultimi 10 frame (ovvero chiamate a funzione) a partire dal punto che ha scatenato l'errore ovvero, a partire dal punto di errore, risale le ultime 10 chiamate a funzione. Se vogliamo modificare questo limite possiamo utilizzare l'opzione --stack-trace-limit 

```bash
$> node --stack-trace-limit=101 app.js
# aumenta a 101 i frame visibili nello stack trace
```