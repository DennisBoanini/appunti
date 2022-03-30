# PROCESS & OPERATING SYSTEM

L'interazione con l'input e output del terminale è nota come standard input/output (STDIO). L'oggetto process espone tre tipi di stream

- process.stdin che è uno stream di tipo Readable per l'input
- process.stdout che è uno stream di tipo Writable per l'output
- process.stderr che è uno stream di tipo Writable per l'output di errore.

Per potersi interfacciare con process.stdin abbiamo la necessità di avere un qualche tipo di input e quindi utilizzeremo un semplice comando che genera dei bytes random in formato esadecimale: `node -p "crypto.randomBytes(100).toString('hex')"`.

```bash
$ node -p "crypto.randomBytes(100).toString('hex')"
0d286cbde8d56952ea62019f6c02ac000516564a12ff6ff944b9f5d32de056abcad5e5f03c9ac74c5090f2498837073
e9b3140431d4d0bb71a493b174f81ecf6f2daf281b5fd65e2f9ac6fd4755791c9fa51fb75a0ec8aba7d38fa8294d829
2f13b610f1
```

Scriviamo un semplice file che stampa la stringa inizializzato

```js
'use strict'
console.log('initialized')

// Risultato
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js
initialized
```

estendiamo il codice per connettere process.stdin a process.stdout

```js
'use strict'
console.log('initialized')
process.stdin.pipe(process.stdout)

// Risultato
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js
initialized
354dea146a80963ee8a8b0a0082dbc50f3f6734837e742c6e67d8ada857946933b8064aa26270432da94024f88f120cf2efac295be6168239793a9d373931418795f39c8c206bac6eb9010742c336f226c1e8e675ecc354706a76de62a4dc35b1977b567
```

Visto che sono stream possiamo utilizzare lo stream Transform per trasformare tutto in uppercase

```js
'use strict'
console.log('initialized')
const { Transform } = require('stream')
const createUppercaseStream = () => {
  return new Transform({
    transform (chunk, enc, next) {
      const uppercased = chunk.toString().toUpperCase()
      next(null, uppercased)
    }
  })
}

const uppercase = createUppercaseStream()

process.stdin.pipe(uppercase).pipe(process.stdout)

// Risultato
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js
initialized
46CA7EF2AE8086F7133AC860A55AB02392E2466F636CFB5B53B6CB5D868CDD44DCEFFE05F3DC564FC51231E965F8FC546DA4F42602B9E77A53F7747D2916B4F879447FF110C8BCEF88A41596F560833E5CA87FBD129532C2BCF305DBE63F1A3D3DC025C4
```

Nel codice sopra è stato utilizzato il metodo pipe anziché pipeline in quando process.stdin, process.stdout e process.stderr sono stream che non finiscono mai (né vanno in errore né si chiudono). Se uno di questi stream finisse causerebbe il crash del processo.

Per vedere se il processo è direttamente connesso al terminale si utilizza process.stdin.isTTY che assume i valori true oppure undefined. Nel codice precedente modifichiamo la prima riga di `console.log('initialized')` in `console.log(process.stdin.isTTY ? 'terminal' : 'piped to')`

```js
'use strict'
console.log(process.stdin.isTTY ? 'terminal' : 'piped to')
const { Transform } = require('stream')
const createUppercaseStream = () => {
  return new Transform({
    transform (chunk, enc, next) {
      const uppercased = chunk.toString().toUpperCase()
      next(null, uppercased)
    }
  })
}

const uppercase = createUppercaseStream()

process.stdin.pipe(uppercase).pipe(process.stdout)

// Risultato
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js
piped to
75DCC9F4622FC9279943511A8BF5832351130E4436AE2432E840F05F6B4FB78560F48676365294922F6DAA29412D8290FC3184A8DB39F09BFF3B77A63825872CF57AF3DF375275987721B99A7DC92872908C6653296BCFF010D56815E988E09C7695535E

// Eseguente invece solamente lo script js
$ node example27.js
terminal
if a process is not piped to, keyboard input can be taken
IF A PROCESS IS NOT PIPED TO, KEYBOARD INPUT CAN BE TAKEN

```

Proviamo adesso a fare il redirect dell'output su un file

```bash
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js > out.txt
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "fs.readFileSync('out.txt').toString()"
piped to
C21F1D37DB5E79BBCF357B5853FB8567F607AD88A92D422B5F51EA98104CC30D83A9C3CBDF725139443B4D1A915A7BA53C4C15DD850DB2AC6001262D2D2E7365C1B44EC36E17EF880495AC51673BA389A9D093E5D75EE36C0A8B5F8611958B8DCA6055B2
```

Sostituiamo `console.log(process.stdin.isTTY ? 'terminal' : 'piped to')` con `process.stderr.write(process.stdin.isTTY ? 'terminal\n' : 'piped to\n')`

```bash
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "crypto.randomBytes(100).toString('hex')" | node example27.js > out.txt
piped to
PS C:\Users\dennis\Desktop\corsi\NodeJS> node -p "fs.readFileSync('out.txt').toString()"
073A08EE599CA4ECDA502CA00CE23FA17946DDF6BE73F5549E12A123D918C65E3AF55FF39AB76C937BBE1E3B1FAEE5B85E0BBCEB149F114170A3C31781C75D882A89BFF2DE12B0CB94033B9FC63335FA7D50211590067E1E111A01715955C2AD0EA33D3F
```

Quando un processo non ha nient'altro da fare, esce, come ad esempio la seguente linea di codice `console.log('exit after this')` che stamperà sul terminale la stringa 'exit after this' e poi termina il processo.
Alcune API hanno un active handle che tengono il processo attivo come ad esempio `net.createServer` che ha un active handle che tiene il processo attivo. Lo stesso setTimeout e setInterval hanno un active handle che impediscono al processo di terminare. Ad esempio il seguente codice stamperà la stringa sul terminale ogni 500ms fino a quando non viene forzato la terminazione del processo con la pressione di CTRL+C

```js
'use strict'
setInterval(() => {
  console.log('this interval is keeping the process open')
}, 500)

// Risultato
dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node example28.js
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
```

Per forzare l'uscita da un processo possiamo utilizzare process.exit. Modifichiamo il codice precedente in questo modo, in modo da terminare il processo dopo 1750ms

```js
'use strict'
setInterval(() => {
  console.log('this interval is keeping the process open')
}, 500)
setTimeout(() => {
  console.log('exit after this')
  process.exit()
}, 1750)

// Risultato
$ node example29.js
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
exit after this
```

Quando si esce da un processo viene impostato uno status code che ha diversi significati in base al SO. 0 è comune ha tutte le piattaforme e uno status code 0 significa che il processo è stato eseguito con successo. Possiamo utilizzare $? per verificare lo status code (oppure echo %ErrorLevel% su cmd Windows o $LastExitCode su powershell).

```bash
$ echo $?0
```

A process.exit è possibile passare uno exit code. Ad esempio possiamo passare un exit code 1 che solitamente significa un fallimento nell'esecuzione.

```js
'use strict'
setInterval(() => {
  console.log('this interval is keeping the process open')
}, 500)
setTimeout(() => {
  console.log('exit after this')
  process.exit(1)
}, 1750)

// Risultato
$ node example30.js
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
exit after this

$ echo $?
1
```

E' possibile anche impostare process.exitCode per impostare uno exit code

```js
'use strict'
setInterval(() => {
  console.log('this interval is keeping the process open')
  process.exitCode = 1
}, 500)
setTimeout(() => {
  console.log('exit after this')
  process.exit()
}, 1750)

// Risultato
$ node example31.js
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
exit after this

$ echo $?
1
```

E' anche possibile mettersi in ascolto sull'evento di exit e fare qualche azione prima di terminare il processo

```js
'use strict'
setInterval(() => {
  console.log('this interval is keeping the process open')
  process.exitCode = 1
}, 500)
setTimeout(() => {
  console.log('exit after this')
  process.exit()
}, 1750)

process.on('exit', (code) => {
  console.log('exiting with code', code)
  setTimeout(() => {
    console.log('this will never happen')
  }, 1)
})

// Risultato
$ node example32.js
this interval is keeping the process open
this interval is keeping the process open
this interval is keeping the process open
exit after this
exiting with code 1

$ echo $?
1
```

L'oggetto process contiene diverse informazioni come ad esempio

- la directory dove il processo sta girando
- la piattaforma dove il processo sta girando
- il suo ID
- le variabili d'ambiente del processo

```js
'use strict'
console.log('Current Directory', process.cwd())
console.log('Process Platform', process.platform)
console.log('Process ID', process.pid)

// Risultato
$ node example33.js
Current Directory C:\Users\dennis\Desktop\corsi\NodeJS
Process Platform win32
Process ID 56744
```

Per le variabili d'ambiente si utilizza process.env

```bash
$ node -p process.env
{
  ACLOCAL_PATH: '/mingw64/share/aclocal:/usr/share/aclocal',
  ALLUSERSPROFILE: 'C:\\ProgramData',
  APPDATA: 'C:\\Users\\dennis\\AppData\\Roaming',
  CHERE_INVOKING: '1',
  ChocolateyInstall: 'C:\\ProgramData\\chocolatey',
  ChocolateyLastPathUpdate: '132451101249070464',
  COMMONPROGRAMFILES: 'C:\\Program Files\\Common Files',
  'CommonProgramFiles(x86)': 'C:\\Program Files (x86)\\Common Files',
  CommonProgramW6432: 'C:\\Program Files\\Common Files',
  COMPUTERNAME: 'PHATE',
  COMSPEC: 'C:\\WINDOWS\\system32\\cmd.exe',
  CONFIG_SITE: '/mingw64/etc/config.site',
  DISPLAY: 'needs-to-be-defined',
  DriverData: 'C:\\Windows\\System32\\Drivers\\DriverData',
  EXEPATH: 'C:\\Program Files\\Git',
  HOME: 'C:\\Users\\dennis',
  HOMEDRIVE: 'C:',
  HOMEPATH: '\\Users\\dennis',
  HOSTNAME: 'Phate',
  INFOPATH: '/usr/local/info:/usr/share/info:/usr/info:/share/info',
  'IntelliJ IDEA': 'C:\\Program Files\\JetBrains\\IntelliJ IDEA 2020.2.2\\bin;',
  LANG: 'it_IT.UTF-8',
  LOCALAPPDATA: 'C:\\Users\\dennis\\AppData\\Local',
  LOGONSERVER: '\\\\PHATE',
  MANPATH: '/mingw64/local/man:/mingw64/share/man:/usr/local/man:/usr/share/man:
/usr/man:/share/man',
  MAVEN_HOME: 'C:\\dev\\apache-maven-3.6.3\\bin',
  MINGW_CHOST: 'x86_64-w64-mingw32',
  MINGW_PACKAGE_PREFIX: 'mingw-w64-x86_64',
  MINGW_PREFIX: '/mingw64',
  MSYSTEM: 'MINGW64',
  MSYSTEM_CARCH: 'x86_64',
  MSYSTEM_CHOST: 'x86_64-w64-mingw32',
  MSYSTEM_PREFIX: '/mingw64',
  NUMBER_OF_PROCESSORS: '12',
  NVM_HOME: 'C:\\Users\\dennis\\AppData\\Roaming\\nvm',
  NVM_SYMLINK: 'C:\\Program Files\\nodejs',
  OLDPWD: '/',
  OneDrive: 'C:\\Users\\denni\\OneDrive',
  OneDriveConsumer: 'C:\\Users\\denni\\OneDrive',
  ORIGINAL_PATH: 'C:\\Program F....',
  ORIGINAL_TEMP: '/tmp',
  ORIGINAL_TMP: '/tmp',
  OS: 'Windows_NT',
  PATH: 'C:\\Users\\dennis\\bi...',
  PATHEXT: '.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.PY;.PYW',
  PKG_CONFIG_PATH: '/mingw64/lib/pkgconfig:/mingw64/share/pkgconfig',
  PLINK_PROTOCOL: 'ssh',
  PROCESSOR_ARCHITECTURE: 'AMD64',
  PROCESSOR_IDENTIFIER: 'Intel64 Family 6 Model 165 Stepping 2, GenuineIntel',
  PROCESSOR_LEVEL: '6',
  PROCESSOR_REVISION: 'a502',
  ProgramData: 'C:\\ProgramData',
  PROGRAMFILES: 'C:\\Program Files',
  'ProgramFiles(x86)': 'C:\\Program Files (x86)',
  ProgramW6432: 'C:\\Program Files',
  PS1: '\\[\\033]0;$TITLEPREFIX:$PWD\\007\\]\\n\\[\\033[32m\\]\\u@\\h \\[\\033[3
5m\\]$MSYSTEM \\[\\033[33m\\]\\w\\[\\033[36m\\]`__git_ps1`\\[\\033[0m\\]\\n$ ',
  PSModulePath: 'C:\\Users\\dennis\\Documents\\WindowsPowerShell\\Modules;C:\\Us
ers\\dennis\\AppData\\Local\\Google\\Cloud SDK\\google-cloud-sdk\\platform\\Powe
rShell',
  PUBLIC: 'C:\\Users\\Public',
  PWD: '/c/Users/dennis/Desktop/corsi/NodeJS',
  'PyCharm Community Edition': 'C:\\Program Files\\JetBrains\\PyCharm Community
Edition 2020.2.3\\bin;',
  SESSIONNAME: 'Console',
  SHELL: 'C:\\Program Files\\Git\\usr\\bin\\bash.exe',
  SHLVL: '1',
  SSH_ASKPASS: '/mingw64/libexec/git-core/git-gui--askpass',
  SYSTEMDRIVE: 'C:',
  SYSTEMROOT: 'C:\\WINDOWS',
  TEMP: 'C:\\Users\\dennis\\AppData\\Local\\Temp',
  TERM_PROGRAM: 'mintty',
  TERM_PROGRAM_VERSION: '3.2.0',
  TMP: 'C:\\Users\\dennis\\AppData\\Local\\Temp',
  TMPDIR: 'C:\\Users\\dennis\\AppData\\Local\\Temp',
  USERDOMAIN: 'PHATE',
  USERDOMAIN_ROAMINGPROFILE: 'PHATE',
  USERNAME: 'dennis',
  USERPROFILE: 'C:\\Users\\dennis',
  WebStorm: 'C:\\Program Files\\JetBrains\\WebStorm 2020.2.2\\bin;',
  WINDIR: 'C:\\WINDOWS',
  ZES_ENABLE_SYSMAN: '1',
  _: '/usr/bin/winpty'
}
```

L'oggett process mette a disposizione diversi metodi per avere informazioni sull'utilizzo di risorse e altri dati statistici. 

Vediamo i metodi process.upTime(), process.cpuUsage e process.memoryUsage.

```js
'use strict'
console.log('Process Uptime', process.uptime())
setTimeout(() => {
  console.log('Process Uptime', process.uptime())
}, 1000)

// Risultato
$ node example34.js
Process Uptime 0.0235087
Process Uptime 1.0433124
```

process.uptime() restituisce il tempo, in secondi con 9 decimali, di esecuzione del process. Nel codice sopra possiamo vedere il tempo di esecuzione del processo nei due punti in cui viene chiamato.

process.cpuUsage restituisce un oggetto con due proprietà: user e system.La proprietà user rappresenta il tempo che il processo Node ha utilizzato utilizzando la CPU, system è il tempo che il kernel ha utilizzato utilizzando la CPU.

```js
'use strict'
const outputStats = () => {
  const uptime = process.uptime()
  const { user, system } = process.cpuUsage()
  console.log(uptime, user, system, (user + system)/1000000)
}

outputStats()

setTimeout(() => {
  outputStats()
  const now = Date.now()
  // make the CPU do some work:
  while (Date.now() - now < 5000) {}
  outputStats()
}, 500)

// Risultato
$ node example35.js
0.0240324 31000 0 0.031
0.5326144 31000 0 0.031
5.5359699 4984000 0 4.984
```

Adesso vediamo process.memoryUsage()

```js
'use strict'
const stats = [process.memoryUsage()]

let iterations = 5

while (iterations--) {
  const arr = []
  let i = 10000
  // make the CPU do some work:
  while (i--) {
    arr.push({[Math.random()]: Math.random()})
  }
  stats.push(process.memoryUsage())
}

console.table(stats)

// Risultato
$ node example36.js
┌─────────┬──────────┬───────────┬──────────┬──────────┬──────────────┐
│ (index) │   rss    │ heapTotal │ heapUsed │ external │ arrayBuffers │
├─────────┼──────────┼───────────┼──────────┼──────────┼──────────────┤
│    0    │ 19169280 │  4603904  │ 3712008  │  220869  │    11146     │
│    1    │ 24186880 │ 10379264  │ 6781640  │  220909  │    11146     │
│    2    │ 31633408 │ 17719296  │ 9336160  │  220909  │    11146     │
│    3    │ 33705984 │ 18505728  │ 12592688 │  220909  │    11146     │
│    4    │ 38285312 │ 20602880  │ 13844728 │  220909  │    11146     │
│    5    │ 42205184 │ 22437888  │ 15208040 │  220909  │    11146     │
└─────────┴──────────┴───────────┴──────────┴──────────┴──────────────┘
```

Per avere informazioni sul sistema operativo possiamo utilizzare il modulo os.

```js
'use strict'
const os = require('os')

console.log('Hostname', os.hostname())
console.log('Home dir', os.homedir())
console.log('Temp dir', os.tmpdir())

// Risultato
$ node example37.js
Hostname Phate
Home dir C:\Users\dennis
Temp dir C:\Users\dennis\AppData\Local\Temp
```

Ci sono due modi possibili per identificare il sistema operativo con il modulo os

- os.platform che restituisce la stessa informazione di process.platform
- os.type che su sistemi non Windows utilizza il comando uname mentre su Windows utilizza il comando ver

```js
'use strict'
const os = require('os')

console.log('platform', os.platform())
console.log('type', os.type())

// Risultato
$ node example38.js
platform win32
type Windows_NT
```

E' possibile recuperare anche informazioni sull'utilizzo del sistema operativo, informazioni come uptime, free memory e total memory. os.uptime restituisce la quantità di tempo di attività del sistema. os.freemem restituisce la quantità di memoria libera mentre os.totalmem restituisce la quantità di memoria disponibile

```js
'use strict'
const os = require('os')

setInterval(() => {
  console.log('system uptime', os.uptime())
  console.log('freemem', os.freemem())
  console.log('totalmem', os.totalmem())
  console.log()
}, 1000)

// Risultato
$ node example39.js
system uptime 2590121
freemem 18671595520
totalmem 34093076480

system uptime 2590122
freemem 18670895104
totalmem 34093076480

system uptime 2590123
freemem 18672410624
totalmem 34093076480

system uptime 2590124
freemem 18680791040
totalmem 34093076480

system uptime 2590125
freemem 18676891648
totalmem 34093076480
```