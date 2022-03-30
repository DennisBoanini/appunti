#  INTERACTING WITH THE FILE SYSTEM

Per lavorare col file system si utilizzano principalmente due moduli del core di NodeJS: fs e path. path lo si utilizza principalmente per lavorare con i path, mentre fs per tutto quello che concerne la lettura, scrittura ecc del file system. Due variabili molto utilizzate sono __filename e __dirname. La prima contiene il path assoluto di dove si trova il file in esecuzione, la seconda contiene il path assoluto della directory che contiene il file attualmente in esecuzione.

```bash
$ node example13.js
current filename C:\Users\dennis\Desktop\corsi\NodeJS\example13.js
current dirname C:\Users\dennis\Desktop\corsi\NodeJS
```

Dato che ogni OS ha il suo modo per rappresentare i percorsi di dove si trovano i file, il metodo join di path ci viene in aiuto ed è utilizzato per creare percorsi di file in base all'OS attualmente in uso

```bash
$ node example14.js
out file: C:\Users\dennis\Desktop\corsi\NodeJS\out.txt
```

Su un altro OS lo script sopra avrebbe restituito un altro percorso. Al path.join possiamo passare quanti argomenti vogliamo senza limite. path.isAbsolute è un metodo di utility che restituisce true se il path è assoluto oppure restituisce false se non lo è. Oltre a questo metodo di utility, tutti gli altri metodi di path possono essere divisi in creatori e deconstruttori. Oltre al path.join altri metodi creatori sono

- path.relative: dati due path assoluti calcola il path relativo
- path.resolve: accetta una serie di stringhe come argomento che rappresentano path. Ogni path rappresenta la navigazione a quell'indirizzo
- path.normalize
- path.format

I metodi "distruttori" sono

- path.parse
- path.extname
- path.dirname
- path.basename

Vediamo il seguente esempio 

```js
'use strict'
const { parse, basename, dirname, extname } = require('path')
console.log('filename parsed:', parse(__filename))
console.log('filename basename:', basename(__filename))
console.log('filename dirname:', dirname(__filename))
console.log('filename extname:', extname(__filename))

// Risultato 
$ node example15.js
filename parsed: {
    root: 'C:\\',
        dir: 'C:\\Users\\dennis\\Desktop\\corsi\\NodeJS',
        base: 'example15.js',
        ext: '.js',
        name: 'example15'
}
filename basename: example15.js
filename dirname: C:\Users\dennis\Desktop\corsi\NodeJS
filename extname: .js
```

Il modulo fs ha API sia di alto livello che di basso livello. fs.open apre e crea un file fornendo un descrittore di tale file. I metodi di alto livello sono forniti in 4 astrazioni diverse

- Sincroni
- Basati su callback
- Basati su promise
- Basati su stream

Tutti i metodi sincroni del modulo fs terminano con Sync. Questi metodi bloccano qualsiasi altra cosa fino a quando non vengono risolti. Sono convenienti per caricare dati quando il programma inizia la sua esecuzione, ma poi dovrebbero essere evitati in quanto bloccano tutto. Vediamo un piccolo esempio

```js
'use strict'
const { readFileSync } = require('fs')
const contents = readFileSync(__filename)
console.log(contents)

// Risultato
$ node example16.js
<Buffer 27 75 73 65 20 73 74 72 69 63 74 27 0a 63 6f 6e 73 74 20 7b 20 72 65 61
64 46 69 6c 65 53 79 6e 63 20 7d 20 3d 20 72 65 71 75 69 72 65 28 27 66 73 27 ..
. 66 more bytes>
```

Il codice sopra legge, in modo sincrono, se stesso e stampa tutto dentro un buffer. Una piccola variante è impostare una codifica, che viene passata come un oggetto di configurazione

```js
'use strict'
const { readFileSync } = require('fs')
const contents = readFileSync(__filename, {encoding: 'utf8'})
console.log(contents)

// Risultato
$ node example17.js
'use strict'
const { readFileSync } = require('fs')
const contents = readFileSync(__filename, {encoding: 'utf8'})
console.log(contents)
```

Il metodo fs.writeFileSync prende come argomenti un path e una stringa e blocca il processo fino a quando il file non è completamente scritto.

```js
'use strict'
const { join } = require('path')
const { readFileSync, writeFileSync } = require('fs')
const contents = readFileSync(__filename, {encoding: 'utf8'})
writeFileSync(join(__dirname, 'out.txt'), contents.toUpperCase())
```

Il codice precedente blocca il processo fino a quando il file out.txt non è completamente scritto. Per aprire il file in modalità append è possibile passare un oggetto di configurazione impostando il valore di flag ad a, come nel seguente codice

```js
'use strict'
const { join } = require('path')
const { readFileSync, writeFileSync } = require('fs')
const contents = readFileSync(__filename, {encoding: 'utf8'})
writeFileSync(join(__dirname, 'out.txt'), contents.toUpperCase(), {
  flag: 'a'
})

//Risultato, eseguendo lo script di prima e questo
$ cat out.txt
'USE STRICT'
CONST { JOIN } = REQUIRE('PATH')
CONST { READFILESYNC, WRITEFILESYNC } = REQUIRE('FS')
CONST CONTENTS = READFILESYNC(__FILENAME, {ENCODING: 'UTF8'})
WRITEFILESYNC(JOIN(__DIRNAME, 'OUT.TXT'), CONTENTS.TOUPPERCASE(), {
  FLAG: 'A'
})
'USE STRICT'
CONST { JOIN } = REQUIRE('PATH')
CONST { READFILESYNC, WRITEFILESYNC } = REQUIRE('FS')
CONST CONTENTS = READFILESYNC(__FILENAME, {ENCODING: 'UTF8'})
WRITEFILESYNC(JOIN(__DIRNAME, 'OUT.TXT'), CONTENTS.TOUPPERCASE(), {
  FLAG: 'A'
})
```

In caso di errori le API *Sync sollevano un'eccezione, per maneggiare gli errori possiamo wrappare tutto in un blocco try/catch.

```bash
$ node -p "fs.chmodSync('out.txt', 0o000)"

dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node example18.js
node:internal/fs/utils:344
    throw err;
    ^

Error: EPERM: operation not permitted, open 'C:\Users\dennis\Desktop\corsi\NodeJ
S\out.txt'
←[90m    at Object.openSync (node:fs:585:3)←[39m
←[90m    at writeFileSync (node:fs:2153:35)←[39m
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example18.js:5:1
)
←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153
:10)←[39m
←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m

←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/r
un_main:81:12)←[39m
←[90m    at node:internal/main/run_main_module:17:47←[39m {
  errno: ←[33m-4048←[39m,
  syscall: ←[32m'open'←[39m,
  code: ←[32m'EPERM'←[39m,
  path: ←[32m'C:\\Users\\dennis\\Desktop\\corsi\\NodeJS\\out.txt'←[39m
}
```

L'equivalente di fs.readFileSync è 

```js
'use strict'
const { readFile } = require('fs')
readFile(__filename, {encoding: 'utf8'}, (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
})
```

Con la funzione readFileSync il processo viene messo in pausa fino a quando la lettura non è completata, col codice precedente invece mentre si sta ancora leggendo si possono eseguire anche altri task. Una volta che la lettura viene completata allora viene eseguita la callback (terzo argomento di readFile) con il risultato della lettura.

Proviamo, in modo asincrono, a scrivere in uppercase il file su out.txt

```js
'use strict'
const { join } = require('path')
const { readFile, writeFile } = require('fs')
readFile(__filename, {encoding: 'utf8'}, (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  const out = join(__dirname, 'out.txt')
  writeFile(out, contents.toUpperCase(), (err) => {
    if (err) { console.error(err) }
  })
})

// Risultato
$ cat out.txt
'USE STRICT'
CONST { JOIN } = REQUIRE('PATH')
CONST { READFILE, WRITEFILE } = REQUIRE('FS')
READFILE(__FILENAME, {ENCODING: 'UTF8'}, (ERR, CONTENTS) => {
  IF (ERR) {
    CONSOLE.ERROR(ERR)
    RETURN
  }
        CONSOLE.LOG(CONTENTS)
  CONST OUT = JOIN(__DIRNAME, 'OUT.TXT')
  WRITEFILE(OUT, CONTENTS.TOUPPERCASE(), (ERR) => {
    IF (ERR) { CONSOLE.ERROR(ERR) }
  })
})
```

fs.promises fornisce la maggior parte dei metodi asincroni disponibili in fs con la differenza che ritornano delle promise anziché delle callback. Quindi anziché prendere readFile e writeFile da fs in questo modo `const { readFile, writeFile } = require('fs')` vengono presi da promises in questo modo `const { readFile, writeFile } = require('fs').promises`.

Vediamo lo stesso codice di prima ma scritto con async/await

```js
'use strict'
const { join } = require('path')
const { readFile, writeFile } = require('fs').promises
async function run () {
  const contents = await readFile(__filename, {encoding: 'utf8'})
  const out = join(__dirname, 'out.txt')
  await writeFile(out, contents.toUpperCase())
}

run().catch(console.error)

// Risultato
$ cat out.txt
'USE STRICT'
CONST { JOIN } = REQUIRE('PATH')
CONST { READFILE, WRITEFILE } = REQUIRE('FS').PROMISES
ASYNC FUNCTION RUN () {
  CONST CONTENTS = AWAIT READFILE(__FILENAME, {ENCODING: 'UTF8'})
  CONST OUT = JOIN(__DIRNAME, 'OUT.TXT')
  AWAIT WRITEFILE(OUT, CONTENTS.TOUPPERCASE())
}

RUN().CATCH(CONSOLE.ERROR)
```

Come si vede il risultato è identico, l'unica differenza è che adesso è asincrono e scritto con async/await.

Il modulo fs ha i metodi fs.createReadStream e fs.createWriteStream che permettono di leggere e scrivere file in chunk. Gli stream sono ideali quando siamo in presenza di file molto grossi che possono essere processati in modo incrementale. 

```js
'use strict'
const { pipeline } = require('stream')
const { join } = require('path')
const { createReadStream, createWriteStream } = require('fs')

pipeline(
  createReadStream(__filename),
  createWriteStream(join(__dirname, 'out.txt')),
  (err) => {
    if (err) {
      console.error(err)
      return
    }
    console.log('finished writing')
  }
)

// Risultato
$ cat out.txt
'use strict'
const { pipeline } = require('stream')
const { join } = require('path')
const { createReadStream, createWriteStream } = require('fs')

pipeline(
  createReadStream(__filename),
  createWriteStream(join(__dirname, 'out.txt')),
  (err) => {
    if (err) {
      console.error(err)
      return
    }
    console.log('finished writing')
  }
)
```

In caso di file molto grossi l'utilizzo di memoria rimane costante in quanto il file viene letto e scritto in piccoli chunk.

Vediamo adesso un esempio precedente che trasforma il file in uppercase utilizzando gli stream

```js
'use strict'
const { pipeline } = require('stream')
const { join } = require('path')
const { createReadStream, createWriteStream } = require('fs')
const { Transform } = require('stream')
const createUppercaseStream = () => {
  return new Transform({
    transform (chunk, enc, next) {
      const uppercased = chunk.toString().toUpperCase()
      next(null, uppercased)
    }
  })
}

pipeline(
  createReadStream(__filename),
  createUppercaseStream(),
  createWriteStream(join(__dirname, 'out.txt')),
  (err) => {
    if (err) {
      console.error(err)
      return
    }
    console.log('finished writing')
  }
)

// Risultato
$ cat out.txt
'USE STRICT'
CONST { PIPELINE } = REQUIRE('STREAM')
CONST { JOIN } = REQUIRE('PATH')
CONST { CREATEREADSTREAM, CREATEWRITESTREAM } = REQUIRE('FS')
CONST { TRANSFORM } = REQUIRE('STREAM')
CONST CREATEUPPERCASESTREAM = () => {
  RETURN NEW TRANSFORM({
    TRANSFORM (CHUNK, ENC, NEXT) {
      CONST UPPERCASED = CHUNK.TOSTRING().TOUPPERCASE()
      NEXT(NULL, UPPERCASED)
    }
  })
}

PIPELINE(
  CREATEREADSTREAM(__FILENAME),
  CREATEUPPERCASESTREAM(),
  CREATEWRITESTREAM(JOIN(__DIRNAME, 'OUT.TXT')),
  (ERR) => {
    IF (ERR) {
      CONSOLE.ERROR(ERR)
      RETURN
    }
    CONSOLE.LOG('FINISHED WRITING')
  }
)
```

Le directory sono tipi speciali di file che contengono collezioni di file. Il modulo fs mette a disposizione diversi modi per leggere una directory

- Sincrono
- Basato su callback
- Basato su promise
- Un iterabile asincrono che eredita da fs.Dir

I pro e contro di ogni punto e le varie caratteristiche sono le stesse della lettura e scrittura di file. 

Supponiamo di avere una directory con all'interno un po' di files e vediamo un esempio di codice che legge i file all'interno della directory in modo sincrono, basato su callback e basato su promise.

```js
'use strict'
const { readdirSync, readdir } = require('fs')
const { readdir: readdirProm } = require('fs').promises

try {
  console.log('sync', readdirSync(__dirname))
} catch (err) {
  console.error(err)
}

readdir(__dirname, (err, files) => {
  if (err) {
    console.error(err)
    return
  }
  console.log('callback', files)
})

async function run () {
  const files = await readdirProm(__dirname)
  console.log('promise', files)
}

run().catch((err) => {
  console.error(err)
})

// Risultato
$ node example24.js
sync [
  'example.js',   'example10.js'
]
callback [
  'example.js',   'example10.js'
]
promise [
  'example.js',   'example10.js'
]
```

La prima parte di codice legge i file all'interno della directory in modo sincrono il che significa che l'esecuzione è ferma fino a quando la lettura non è terminata. La seconda parte di codice invece utilizza le callback ovvero quando la lettura è terminata viene eseguita la funzione passata come argomento a readdir.

I metadata di files possono essere ottenuti in diversi modi

- fs.stat, fs.statSync e fs.promises.stat
- fs.lstat, fs.lstatSync e fs.promises.lstat

La sostanziale differenza fra stat e lstat è che stat segue i link simbolici mentre lstat prende i metadata dei link simbolici.

Tutti i metodi sopra elencati ritornano un'istanza di fs.Stat la quale contiene diverse proprietà e metodi per accedere ai metadata dei file.

Vediamo un esempio di codice che legge il contenuto di una directory e per ogni elemento dice se è una directory o no

```js
'use strict'
const { readdirSync, statSync } = require('fs')

const files = readdirSync('.')

for (const name of files) {
  const stat = statSync(name)
  const typeLabel = stat.isDirectory() ? 'dir: ' : 'file: '
  console.log(typeLabel, name)
}

// Risultato
$ node example25.js
file:  example.js
file:  example10.js
dir:  labs
dir:  my_package
file:  out
file:  out.txt
```

Vediamo un po' di statistiche sull'orario. Ci sono 4 tipologie di informazioni che possiamo avere

- Access Time
- Change Time
- Modified Time
- Birth Time

```js
'use strict'
const { readdirSync, statSync } = require('fs')

const files = readdirSync('.')

for (const name of files) {
  const stat = statSync(name)
  const typeLabel = stat.isDirectory() ? 'dir: ' : 'file: '
  const { atime, birthtime, ctime, mtime } = stat
  console.group(typeLabel, name)
  console.log('atime:', atime.toLocaleString())
  console.log('ctime:', ctime.toLocaleString())
  console.log('mtime:', mtime.toLocaleString())
  console.log('birthtime:', birthtime.toLocaleString())
  console.groupEnd()
  console.log()
}

// Risultato
$ node example26.js
file:  example.js
  atime: 22/3/2022, 22:26:18
  ctime: 15/3/2022, 21:55:03
  mtime: 15/3/2022, 21:55:03
  birthtime: 15/3/2022, 00:30:51

dir:  labs
  atime: 22/3/2022, 16:40:04
  ctime: 8/3/2022, 01:28:36
  mtime: 8/3/2022, 01:28:36
  birthtime: 8/3/2022, 01:28:35

dir:  my_package
  atime: 23/3/2022, 11:29:25
  ctime: 12/3/2022, 01:57:16
  mtime: 12/3/2022, 01:57:16
  birthtime: 11/3/2022, 22:19:46

file:  out
  atime: 22/3/2022, 22:26:18
  ctime: 21/3/2022, 22:43:12
  mtime: 21/3/2022, 22:43:12
  birthtime: 21/3/2022, 22:43:12

file:  out.txt
  atime: 22/3/2022, 22:48:46
  ctime: 22/3/2022, 22:48:43
  mtime: 22/3/2022, 22:48:43
  birthtime: 22/3/2022, 21:45:02
```