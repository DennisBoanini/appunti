# NODE'S MODULE SYSTEM

Supponiamo di avere un file index.js con questo contenuto

```js
'use strict'
const pino = require('pino')
const logger = pino()
logger.info('my-package started')
process.stdin.resume()
```

possiamo vedere in seconda linea come, per caricare il modulo pino, utilizziamo la parola chiave require. Alla funzione require viene passato il nome del namespace del package che sarà cercato all'interno della directory node_modules e poi restituisce il nome del modulo esportato che viene assegnato alla costante pino che abbiamo dichiarato. Nel nostro caso di esempio il modulo pino esporta una funzione che noi utilizziamo a riga 3 per instanziare il logger. require non ritorna solamente funzioni come nel nostro caso, ma può ritornare qualsiasi cosa.

Creiamo un file chiamato format.js con questo contenuto

```js
'use strict'

const upper = (str) => {
  if (typeof str === 'symbol') str = str.toString()
  str += ''
  return str.toUpperCase()
}

module.exports = { upper: upper}
```

Abbiamo creato una funzione la quale prende in input una stringa e la trasforma in uppercase. Tutto quello che assegniamo a module.exports rappresenta il valore ritornato quando il modulo viene richiesto (attraverso il require). A questo punto il file format.js può essere caricato all'interno di index.js

```js
'use strict'
const pino = require('pino')
const format = require('./format')
const logger = pino()
logger.info(format.upper('my-package started'))
process.stdin.resume()
```

Quindi quello che succede è che require('./format') restituisce il valore di module.exports di format.js che è un oggetto con un metodo chiamato upper.

Lo script start presente nel package.json esegue node index.js e quando un file è chiamato con node questo rappresenta il punto di ingresso per il nostro programma. In alcune situazioni vorremmo che il nostro modulo sia in grado di lavorare sia come programma che come modulo che può essere caricato all'interno di altri moduli. E' possibile capire quando un file è un modulo principale o meno. Modifichiamo il file index.js in questo modo 

```js
'use strict'
const format = require('./format')

if (require.main === module) {
    const pino = require('pino')
    const logger = pino()
    logger.info(format.upper('my-package started'))
    process.stdin.resume()
} else {
    const reverseAndUpper = (str) => {
        return format.upper(str).split('').reverse().join('')
    }
    module.exports = reverseAndUpper
}
```

Adesso il file index.js può operare in due modi diversi: 

- se caricato come modulo allora esporta una funzione che fa il reverse di una stringa e la mette in uppercase
```js
$ node -p "require('./index.js')('hello')"
OLLEH
```

- se invece viene eseguito con node, allora fa il suo comportamento originale

```js
$ node index.js
{"level":30,"time":1647035996550,"pid":45608,"hostname":"Phate","msg":"MY-PACKAGE STARTED"}
```

I moduili EcmaScript (ESM) sono stati introdotti da EcmaScript 2015 (conosciuto come ES6). Una delle principali caratteristiche è la possibilità di essere staticamente analizzabili. La principale differenza con i moduli CJM (CommonJS) è che CJS carica tutti i moduli in modo sincrono mentre ESM li carica in modo asincrono. Un'applicazione Node può contenere sia moduli ESM che moduli CJS.

Proviamo a convertire il file format.js da CJS a ESM. CJS modifica module.exports mentre ESM ha una sintassi nativa e per creare un export si utilizza la parola chiave export davanti a quello che vogliamo esportare.

```js
export const upper = (str) => {
  if (typeof str === 'symbol') str = str.toString()
  str += ''
  return str.toUpperCase()
}
```

non c'è più bisogno nemmeno di 'use strict' in quanto ESM lo utilizza già di suo. Se però a questo punto provo ad eseguire node index.js da terminale ottengo il seguente risultato

```js
$ node index.js
node:internal/modules/cjs/loader:936
  throw err;
  ^

Error: Cannot find module './format'
Require stack:
- C:\Users\dennis\Desktop\corsi\NodeJS\my_package\index.js
←[90m    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:933:15)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:778:27)←[39m
←[90m    at Module.require (node:internal/modules/cjs/loader:1005:19)←[39m
←[90m    at require (node:internal/modules/cjs/helpers:102:18)←[39m
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\my_package\index.js:2:16)
←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)←[39m
←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m
←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)←[39m {
  code: ←[32m'MODULE_NOT_FOUND'←[39m,
  requireStack: [ ←[32m'C:\\Users\\dennis\\Desktop\\corsi\\NodeJS\\my_package\\index.js'←[39m ]
}
```

questo succede perchè require non è in grado di risolvere automaticamente un filename senza estensione ('./format').

La differenza fra caricamento sincrono e asincrono è fondamentale in quanto mentre ESM può importare CJS, CJS non può importare ESM dal momento che romperebbe la sincronicità. Per fare in modo che entrambi i moduli riescano a cooperare essi devono esperre una interfaccia CJS. E' comunque possibile caricare in modo asincrono un modulo ESM per essere utilizzato all'interno di un modulo CJS. Modifichiamo index.js in questo modo 

```js
'use strict'

if (require.main === module) {
  const pino = require('pino')
  const logger = pino()
  import('./format.mjs').then((format) => {
    logger.info(format.upper('my-package started'))
    process.stdin.resume()
  }).catch((err) => {
    console.error(err)
    process.exit(1)
  })
} else {
  let format = null
  const reverseAndUpper = async (str) => {
    format = format || await import('./format.mjs')
    return format.upper(str).split('').reverse().join('')
  }
  module.exports = reverseAndUpper
}
```

Per utilizzare ESM di default lo si dichiara all'interno del package.json

```js
{
    "name": "my-package",
        "version": "1.0.0",
        "main": "index.js",
        "type": "module",
        "scripts": {
        "start": "node index.js",
            "test": "echo \"Error: no test specified\" && exit 1",
            "lint": "standard"
    },
    "author": "",
        "license": "ISC",
        "keywords": [],
        "description": "",
        "dependencies": {
        "pino": "^7.6.2"
    },
    "devDependencies": {
        "standard": "^16.0.4"
    }
}
```
la coppia "type": "module" ci dice che stiamo utilizzando ESM.

Rinominamo format.mjs a format.js e modifichiamo index.js in questo modo

```js
import { realpath } from 'fs/promises'
import url from 'url'
import * as format from './format.js'

const isMain = process.argv[1] &&
 await realpath(fileURLToPath(import.meta.url)) ===
 await realpath(process.argv[1])

if (isMain) {
  const { default: pino } = await import('pino')
  const logger = pino()
  logger.info(format.upper('my-package started'))
  process.stdin.resume()
}

export default (str) => {
  return format.upper(str).split('').reverse().join('')
}
```