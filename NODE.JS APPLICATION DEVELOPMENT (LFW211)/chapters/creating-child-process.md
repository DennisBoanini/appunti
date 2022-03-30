# CREATING CHILD PROCESSES

Per creare nuovi processi (processi figli) all'interno del processo attualmente in esecuzione viene utilizzato il modulo child_process. Il nuovo processo può essere un qualsiasi eseguibile scritto in qualsiasi linguaggio di programmazione.

Il modulo child_process ha i seguenti metodi, e tutti possono creare un nuovo processo:

- exec & execSync
- spawn & spawnSync
- execFile & execFileSync
- fork

execFile e execFileSync sono semplici varianti di exec e execSync. Anziché eseguire il comando all'interno della shell, tenta di eseguire il comando all'interno di un binario eseguibile.

fork è una specializzazione di spawn e di default crea un nuovo processo Node del file JavaScript attualmente in esecuzione.

child_process.execSync è il modo più semplice per eseguire un comando

```js
'use strict'
const { execSync } = require('child_process')
const output = execSync(
  `node -e "console.log('subprocess stdio output')"`
)
console.log(output.toString())

// Risultato 
$ node example40.js
subprocess stdio output
```

Il metodo execSync restituisce un buffer che contiene l'output del comando (output preso da STDOUT). Se al posto di console.log si usa console.error allora il processo figlio scriverà su STDERR. Di default execSync redirige l'output STDERR al STDERR del padre.

Nel codice precedente abbiamo eseguito un comando node, ma è possibile eseguire qualsiasi tipo di comando, come ad esempio nel seguente codice che utilizza i comandi ls (o dir) per fare l'elenco dei file all'interno della directory.

```js
'use strict'
const { execSync } = require('child_process')
const cmd = process.platform === 'win32' ? 'dir' : 'ls'
const output = execSync(cmd)
console.log(output.toString())

// Risultato
$ node example41.js
 Il volume nell'unit� C � OS
 Numero di serie del volume: 2CED-1A92

 Directory di C:\Users\dennis\Desktop\corsi\NodeJS

24/03/2022  16:33    <DIR>          .
08/03/2022  01:28    <DIR>          ..
15/03/2022  21:55               882 example.js
22/03/2022  14:53               253 example10.js
08/03/2022  01:28    <DIR>          labs
12/03/2022  01:57    <DIR>          my_package
21/03/2022  22:43                27 out
23/03/2022  23:26               406 out.txt
              43 File         12.666 byte
               4 Directory  336.550.125.568 byte disponibili
```

Se volessimo eseguire node come un processo figlio, è meglio riferisti all'intero path del processo corrente di Node e questo può essere trovato attraverso il comando `process.execPath`

```bash
$ node -p process.execPath
C:\Program Files\nodejs\node.exe
```

Vediamo process.execPath in azione all'interno di uno script

```js
'use strict'
const { execSync } = require('child_process')
const output = execSync(
  `"${process.execPath}" -e "console.error('subprocess stdio output')"`
)
console.log(output.toString())

// Risultato
$ node example42.js
subprocess stdio output
```

Se il sottoprocesso termina con un codice diverso da 0, la funzione execSync solleva un errore

```js
'use strict'
const { execSync } = require('child_process')

try {
  execSync(`"${process.execPath}" -e "process.exit(1)"`)
} catch (err) {
  console.error('CAUGHT ERROR:', err)
}

// Risultato
$ node example43.js
CAUGHT ERROR: Error: Command failed: "C:\Program Files\nodejs\node.exe" -e "process.exit(1)"
    at checkExecSyncError (node:child_process:826:11)
    at execSync (node:child_process:900:15)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example43.js:5:3
)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_ma
in:81:12)
    at node:internal/main/run_main_module:17:47 {
  status: 1,
  signal: null,
  output: [ null, <Buffer >, <Buffer > ],
  pid: 69588,
  stdout: <Buffer >,
  stderr: <Buffer >
}
```

L'oggetto loggato nel catch ha diverse proprietà. La proprietà status ha valore 1 in quando deriva dal fatto che abbiamo invocato l'istruzione process.exit(1). In scenari in cui il codice di uscita è diverso da 0 la proprietà stderr è molto utile. output è un array i cui indici corrispondono ai file descriptor I/O. Ad esempio il file descriptor di STDERR è 2 quindi err.stderr è uguale a err.output[2]

Modifichiamo leggermente il codice precedente per lanciare un errore

```js
'use strict'
const { execSync } = require('child_process')

try {
  execSync(`"${process.execPath}" -e "throw Error('kaboom')"`)
} catch (err) {
  console.error('CAUGHT ERROR:', err)
}

// Risultato
$ node example44.js
[eval]:1
throw Error('kaboom')
^

Error: kaboom
    at [eval]:1:7
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:81:19
    at [eval]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_string:27:3
CAUGHT ERROR: Error: Command failed: "C:\Program Files\nodejs\node.exe" -e "throw Error('kaboom')"
[eval]:1
throw Error('kaboom')
^

Error: kaboom
    at [eval]:1:7
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:81:19
    at [eval]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_string:27:3

    at checkExecSyncError (node:child_process:826:11)
    at execSync (node:child_process:900:15)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example44.js:5:3
)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_ma
in:81:12)
    at node:internal/main/run_main_module:17:47 {
  status: 1,
  signal: null,
  output: [
    null,
    <Buffer >,
    <Buffer 5b 65 76 61 6c 5d 3a 31 0d 0a 74 68 72 6f 77 20 45 72 72 6f 72 28 27
 6b 61 62 6f 6f 6d 27 29 0d 0a 5e 0d 0a 0d 0a 45 72 72 6f 72 3a 20 6b 61 62 6f 6
f ... 297 more bytes>
  ],
  pid: 71216,
  stdout: <Buffer >,
  stderr: <Buffer 5b 65 76 61 6c 5d 3a 31 0d 0a 74 68 72 6f 77 20 45 72 72 6f 72
 28 27 6b 61 62 6f 6f 6d 27 29 0d 0a 5e 0d 0a 0d 0a 45 72 72 6f 72 3a 20 6b 61 6
2 6f 6f ... 297 more bytes>
}
```

La prima parte dell'output, dove viene scritto CAUGHT ERROR, rappresenta l'output del sottoprocesso. Lo stesso oggetto è contenuto in err.stderr e err.output[2]. La stampa dell'errore sembra doppia ma la prima stampa rappresenta la funzione chiamata all'interno del sottoprocesso, mentre la seconda rappresenta la funzione chiamata nel processo padre.

Il metodo exec funziona allo stesso modo di execSync con la differenza che exec splitta STDOUT e STDERR passandoli entrambi come argomenti ad una funzione di callback

```js
'use strict'
const { exec } = require('child_process')

exec(
  `"${process.execPath}" -e "console.log('A');console.error('B')"`,
  (err, stdout, stderr) => {
    console.log('err', err)
    console.log('subprocess stdout: ', stdout.toString())
    console.log('subprocess stderr: ', stderr.toString())
  }
)

// Risultato
$ node example45.js
err null
subprocess stdout:  A

subprocess stderr:  B
```

A differenza di exec e execSync, i metodi spawn e spawnSync prendono come primo argomento il path completo dell'eseguibile (node o altro) e come secondo argomento prende un array di flag

```js
'use strict'
const { spawnSync } = require('child_process')
const result = spawnSync(
  process.execPath,
  ['-e', `console.log('subprocess stdio output')`]
)
console.log(result)

// Risultato
$ node example46.js
{
  status: 0,
  signal: null,
  output: [
    null,
    <Buffer 73 75 62 70 72 6f 63 65 73 73 20 73 74 64 69 6f 20 6f 75 74 70 75 74
 0a>,
    <Buffer >
  ],
  pid: 44332,
  stdout: <Buffer 73 75 62 70 72 6f 63 65 73 73 20 73 74 64 69 6f 20 6f 75 74 70
 75 74 0a>,
  stderr: <Buffer >
}
```

Mentre execSync ritornava un buffer contenente l'output del processo figlio, spawnSync ritorna un oggetto che contiene le informazioni sul processo che è stato "spawned", questo oggetto viene assegnato alla costante result e stampata. 

A differenza di execSync, spawnSync non ha bisogno di essere wrappato in un try/catch; se spawnSync termina con un codice diverse da 0, non solleva nessuna eccezione

```js
'use strict'
const { spawnSync } = require('child_process')
const result = spawnSync(process.execPath, [`-e`, `process.exit(1)`])
console.log(result)

// Risultato
$ node example47.js
{
  status: 1,
  signal: null,
  output: [ null, <Buffer >, <Buffer > ],
  pid: 43064,
  stdout: <Buffer >,
  stderr: <Buffer >
}
```

Vediamo un esempio del metodo spawn

```js
'use strict'
const { spawn } = require('child_process')

const sp = spawn(
  process.execPath,
  [`-e`, `console.log('subprocess stdio output')`]
)

console.log('pid is', sp.pid)

sp.stdout.pipe(process.stdout)

sp.on('close', (status) => {
  console.log('exit status was', status)
})

// Risultato
$ node example48.js
pid is 43768
subprocess stdio output
exit status was 0
```

Il metodo spawn ritorna un'istanza di ChildProcess che viene assegnata a sp.

Quando si utilizzano spawn e spawnSync un oggetto di configurazione può essere passato come terzo argomento (o come secondo argomento se stiamo utilizzando exec o execSync). Due delle opzioni che possiamo passare cono cwd e env che vengono utilizzate per controllare l'ambiente di esecuzione dei processi figli. 

Di default i processi figli ereditano le variabili d'ambiente del processo padre

```js
'use strict'
const { spawn } = require('child_process')

process.env.A_VAR_WE = 'JUST SET'
const sp = spawn(process.execPath, ['-p', 'process.env'])
sp.stdout.pipe(process.stdout)

// Risultato
$ node example49.js
{
  ACLOCAL_PATH: '/mingw64/share/aclocal:/usr/share/aclocal',
  ALLUSERSPROFILE: 'C:\\ProgramData',
  APPDATA: 'C:\\Users\\dennis\\AppData\\Roaming',
  A_VAR_WE: 'JUST SET',
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
  ORIGINAL_PATH: 'C:\\Program Files\\Git\\mingw64\\bin;C:\\Program Files\\Git\\u
sr\\bin;C:\\Users\\dennis\\bin;C:\\Program Files\\Common Files\\Oracle\\Java\\ja
vapath;C:\\Program Files (x86)\\Common Files\\Oracle\\Java\\javapath;C:\\Python3
8\\Scripts;C:\\Python38;C:\\Windows\\system32;C:\\Windows;C:\\Windows\\System32\
\Wbem;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;C:\\Windows\\System32\\Open
SSH;C:\\Program Files (x86)\\NVIDIA Corporation\\PhysX\\Common;C:\\Program Files
\\NVIDIA Corporation\\NVIDIA NvDLISR;C:\\ProgramData\\chocolatey\\bin;C:\\Progra
m Files\\Git\\cmd;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\no
dejs;C:\\Program Files (x86)\\Yarn\\bin;C:\\Program Files\\PostgreSQL\\13\\bin;C
:\\Program Files\\MySQL\\MySQL Server 8.0\\bin;C:\\Program Files\\Kubernetes\\Mi
nikube;C:\\Program Files\\MongoDB\\Server\\4.4\\bin;C:\\Program Files\\MongoDB\\
Tools\\100\\bin;C:\\WINDOWS\\system32;C:\\WINDOWS;C:\\WINDOWS\\System32\\Wbem;C:
\\WINDOWS\\System32\\WindowsPowerShell\\v1.0;C:\\WINDOWS\\System32\\OpenSSH;C:\\
Program Files\\Docker\\Docker\\resources\\bin;C:\\ProgramData\\DockerDesktop\\ve
rsion-bin;C:\\Users\\dennis\\AppData\\Local\\Microsoft\\WindowsApps;C:\\Program
Files\\JetBrains\\WebStorm 2020.2.2\\bin;C:\\Program Files\\JetBrains\\IntelliJ
IDEA 2020.2.2\\bin;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\n
odejs;C:\\Program Files\\JetBrains\\PyCharm Community Edition 2020.2.3\\bin;C:\\
dev\\apache-maven-3.6.3\\bin;C:\\dev;C:\\Users\\dennis\\AppData\\Local\\Yarn\\bi
n;C:\\Users\\dennis\\AppData\\Local\\Programs\\Microsoft VS Code\\bin;C:\\Exerci
sm;C:\\Users\\dennis\\AppData\\Local\\Google\\Cloud SDK\\google-cloud-sdk\\bin;C
:\\Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resources\\resource
s\\win32\\bin;C:\\Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\reso
urces\\resources\\linux\\bin',
  ORIGINAL_TEMP: '/tmp',
  ORIGINAL_TMP: '/tmp',
  OS: 'Windows_NT',
  PATH: 'C:\\Users\\dennis\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program
 Files\\Git\\usr\\local\\bin;C:\\Program Files\\Git\\usr\\bin;C:\\Program Files\
\Git\\usr\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program Files\\Git\\usr\
\bin;C:\\Users\\dennis\\bin;C:\\Program Files\\Common Files\\Oracle\\Java\\javap
ath;C:\\Program Files (x86)\\Common Files\\Oracle\\Java\\javapath;C:\\Python38\\
Scripts;C:\\Python38;C:\\Windows\\system32;C:\\Windows;C:\\Windows\\System32\\Wb
em;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;C:\\Windows\\System32\\OpenSSH
;C:\\Program Files (x86)\\NVIDIA Corporation\\PhysX\\Common;C:\\Program Files\\N
VIDIA Corporation\\NVIDIA NvDLISR;C:\\ProgramData\\chocolatey\\bin;C:\\Program F
iles\\Git\\cmd;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\nodej
s;C:\\Program Files (x86)\\Yarn\\bin;C:\\Program Files\\PostgreSQL\\13\\bin;C:\\
Program Files\\MySQL\\MySQL Server 8.0\\bin;C:\\Program Files\\Kubernetes\\Minik
ube;C:\\Program Files\\MongoDB\\Server\\4.4\\bin;C:\\Program Files\\MongoDB\\Too
ls\\100\\bin;C:\\WINDOWS\\system32;C:\\WINDOWS;C:\\WINDOWS\\System32\\Wbem;C:\\W
INDOWS\\System32\\WindowsPowerShell\\v1.0;C:\\WINDOWS\\System32\\OpenSSH;C:\\Pro
gram Files\\Docker\\Docker\\resources\\bin;C:\\ProgramData\\DockerDesktop\\versi
on-bin;C:\\Users\\dennis\\AppData\\Local\\Microsoft\\WindowsApps;C:\\Program Fil
es\\JetBrains\\WebStorm 2020.2.2\\bin;C:\\Program Files\\JetBrains\\IntelliJ IDE
A 2020.2.2\\bin;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\node
js;C:\\Program Files\\JetBrains\\PyCharm Community Edition 2020.2.3\\bin;C:\\dev
\\apache-maven-3.6.3\\bin;C:\\dev;C:\\Users\\dennis\\AppData\\Local\\Yarn\\bin;C
:\\Users\\dennis\\AppData\\Local\\Programs\\Microsoft VS Code\\bin;C:\\Exercism;
C:\\Users\\dennis\\AppData\\Local\\Google\\Cloud SDK\\google-cloud-sdk\\bin;C:\\
Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resources\\resources\\
win32\\bin;C:\\Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resourc
es\\resources\\linux\\bin;C:\\Program Files\\Git\\usr\\bin\\vendor_perl;C:\\Prog
ram Files\\Git\\usr\\bin\\core_perl',
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

Il codice di esempio sopra crea un processo figlio che esegue il comando node con -p in modo da stampare le variabili d'ambiente ed uscire. stdout del processo figlio viene messo in pipe con stdout del padre, in modo da stampare il risultato sul terminale. Tutte le variabili d'ambiente stampate sono quelle del processo figlio che abbiamo creato.

Per sovrascrivere le variabili d'ambiente del processo padre è necessario passare l'opzione env

```js
'use strict'

const { spawn } = require('child_process')

process.env.A_VAR_WE = 'JUST SET'

const sp = spawn(process.execPath, ['-p', 'process.env'], {
  env: {SUBPROCESS_SPECIFIC: 'ENV VAR'}
})

sp.stdout.pipe(process.stdout)

// Risultato
$ node example50.js
{
  HOMEDRIVE: 'C:',
  HOMEPATH: '\\Users\\dennis',
  LOGONSERVER: '\\\\PHATE',
  PATH: 'C:\\Users\\dennis\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program
 Files\\Git\\usr\\local\\bin;C:\\Program Files\\Git\\usr\\bin;C:\\Program Files\
\Git\\usr\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program Files\\Git\\usr\
\bin;C:\\Users\\dennis\\bin;C:\\Program Files\\Common Files\\Oracle\\Java\\javap
ath;C:\\Program Files (x86)\\Common Files\\Oracle\\Java\\javapath;C:\\Python38\\
Scripts;C:\\Python38;C:\\Windows\\system32;C:\\Windows;C:\\Windows\\System32\\Wb
em;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;C:\\Windows\\System32\\OpenSSH
;C:\\Program Files (x86)\\NVIDIA Corporation\\PhysX\\Common;C:\\Program Files\\N
VIDIA Corporation\\NVIDIA NvDLISR;C:\\ProgramData\\chocolatey\\bin;C:\\Program F
iles\\Git\\cmd;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\nodej
s;C:\\Program Files (x86)\\Yarn\\bin;C:\\Program Files\\PostgreSQL\\13\\bin;C:\\
Program Files\\MySQL\\MySQL Server 8.0\\bin;C:\\Program Files\\Kubernetes\\Minik
ube;C:\\Program Files\\MongoDB\\Server\\4.4\\bin;C:\\Program Files\\MongoDB\\Too
ls\\100\\bin;C:\\WINDOWS\\system32;C:\\WINDOWS;C:\\WINDOWS\\System32\\Wbem;C:\\W
INDOWS\\System32\\WindowsPowerShell\\v1.0;C:\\WINDOWS\\System32\\OpenSSH;C:\\Pro
gram Files\\Docker\\Docker\\resources\\bin;C:\\ProgramData\\DockerDesktop\\versi
on-bin;C:\\Users\\dennis\\AppData\\Local\\Microsoft\\WindowsApps;C:\\Program Fil
es\\JetBrains\\WebStorm 2020.2.2\\bin;C:\\Program Files\\JetBrains\\IntelliJ IDE
A 2020.2.2\\bin;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\node
js;C:\\Program Files\\JetBrains\\PyCharm Community Edition 2020.2.3\\bin;C:\\dev
\\apache-maven-3.6.3\\bin;C:\\dev;C:\\Users\\dennis\\AppData\\Local\\Yarn\\bin;C
:\\Users\\dennis\\AppData\\Local\\Programs\\Microsoft VS Code\\bin;C:\\Exercism;
C:\\Users\\dennis\\AppData\\Local\\Google\\Cloud SDK\\google-cloud-sdk\\bin;C:\\
Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resources\\resources\\
win32\\bin;C:\\Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resourc
es\\resources\\linux\\bin;C:\\Program Files\\Git\\usr\\bin\\vendor_perl;C:\\Prog
ram Files\\Git\\usr\\bin\\core_perl',
  SUBPROCESS_SPECIFIC: 'ENV VAR',
  SYSTEMDRIVE: 'C:',
  SYSTEMROOT: 'C:\\WINDOWS',
  TEMP: 'C:\\Users\\dennis\\AppData\\Local\\Temp',
  USERDOMAIN: 'PHATE',
  USERNAME: 'dennis',
  USERPROFILE: 'C:\\Users\\dennis',
  WINDIR: 'C:\\WINDOWS'
}
```

Un'altra opzione che è possibile passare quando creiamo un processo figlio è cwd.

```js
'use strict'
const { IS_CHILD } = process.env

if (IS_CHILD) {
  console.log('Subprocess cwd:', process.cwd())
  console.log('env', process.env)
} else {
  const { parse } = require('path')
  const { root } = parse(process.cwd())
  const { spawn } = require('child_process')
  const sp = spawn(process.execPath, [__filename], {
    cwd: root,
    env: {IS_CHILD: '1'}
  })

  sp.stdout.pipe(process.stdout)
}

// Risultato
$ node example51.js
Subprocess cwd: C:\
env {
  HOMEDRIVE: 'C:',
  HOMEPATH: '\\Users\\dennis',
  IS_CHILD: '1',
  LOGONSERVER: '\\\\PHATE',
  PATH: 'C:\\Users\\dennis\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program
 Files\\Git\\usr\\local\\bin;C:\\Program Files\\Git\\usr\\bin;C:\\Program Files\
\Git\\usr\\bin;C:\\Program Files\\Git\\mingw64\\bin;C:\\Program Files\\Git\\usr\
\bin;C:\\Users\\dennis\\bin;C:\\Program Files\\Common Files\\Oracle\\Java\\javap
ath;C:\\Program Files (x86)\\Common Files\\Oracle\\Java\\javapath;C:\\Python38\\
Scripts;C:\\Python38;C:\\Windows\\system32;C:\\Windows;C:\\Windows\\System32\\Wb
em;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;C:\\Windows\\System32\\OpenSSH
;C:\\Program Files (x86)\\NVIDIA Corporation\\PhysX\\Common;C:\\Program Files\\N
VIDIA Corporation\\NVIDIA NvDLISR;C:\\ProgramData\\chocolatey\\bin;C:\\Program F
iles\\Git\\cmd;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\nodej
s;C:\\Program Files (x86)\\Yarn\\bin;C:\\Program Files\\PostgreSQL\\13\\bin;C:\\
Program Files\\MySQL\\MySQL Server 8.0\\bin;C:\\Program Files\\Kubernetes\\Minik
ube;C:\\Program Files\\MongoDB\\Server\\4.4\\bin;C:\\Program Files\\MongoDB\\Too
ls\\100\\bin;C:\\WINDOWS\\system32;C:\\WINDOWS;C:\\WINDOWS\\System32\\Wbem;C:\\W
INDOWS\\System32\\WindowsPowerShell\\v1.0;C:\\WINDOWS\\System32\\OpenSSH;C:\\Pro
gram Files\\Docker\\Docker\\resources\\bin;C:\\ProgramData\\DockerDesktop\\versi
on-bin;C:\\Users\\dennis\\AppData\\Local\\Microsoft\\WindowsApps;C:\\Program Fil
es\\JetBrains\\WebStorm 2020.2.2\\bin;C:\\Program Files\\JetBrains\\IntelliJ IDE
A 2020.2.2\\bin;C:\\Users\\dennis\\AppData\\Roaming\\nvm;C:\\Program Files\\node
js;C:\\Program Files\\JetBrains\\PyCharm Community Edition 2020.2.3\\bin;C:\\dev
\\apache-maven-3.6.3\\bin;C:\\dev;C:\\Users\\dennis\\AppData\\Local\\Yarn\\bin;C
:\\Users\\dennis\\AppData\\Local\\Programs\\Microsoft VS Code\\bin;C:\\Exercism;
C:\\Users\\dennis\\AppData\\Local\\Google\\Cloud SDK\\google-cloud-sdk\\bin;C:\\
Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resources\\resources\\
win32\\bin;C:\\Users\\dennis\\AppData\\Local\\Programs\\Rancher Desktop\\resourc
es\\resources\\linux\\bin;C:\\Program Files\\Git\\usr\\bin\\vendor_perl;C:\\Prog
ram Files\\Git\\usr\\bin\\core_perl',
  SYSTEMDRIVE: 'C:',
  SYSTEMROOT: 'C:\\WINDOWS',
  TEMP: 'C:\\Users\\dennis\\AppData\\Local\\Temp',
  USERDOMAIN: 'PHATE',
  USERNAME: 'dennis',
  USERPROFILE: 'C:\\Users\\dennis',
  WINDIR: 'C:\\WINDOWS'
}
```

Nell'esempio di codice sopra, lo stesso file viene eseguito due volte. Una volta come processo padre e una volta come processo figlio. Abbiamo fatto lo spawn del processo figlio passando __filename fra gli argomenti passati al comando spawn, questo significa che il processo figlio esegue node col path del file attuale.

Abbiamo passato l'opzione env al comando spawn con una proprietè IS_CHILD settata a 1, in questo modo quando il sottoprocesso viene caricato si entra nel blocco if. Mentre nel processo padre process.env.IS_CHILD risulta essere undefined, quando il processo padre viene eseguito si entra nell'else dove il viene fatto lo spawn del processo figlio.

I mnetodi asincroni utilizzati per creare processi (execSync e spawnSync) ritornano un'istanza di ChildProcess che ha tre stream che rappresentano l'I/O del sottoprocesso: stdin, stdout e stderr. Questo comportamente può anche essere alterato volendo.

```js
'use strict'
const { spawn } = require('child_process')
const sp = spawn(
  process.execPath,
  [
   '-e',
   `console.error('err output'); process.stdin.pipe(process.stdout)`
  ],
  { stdio: ['pipe', 'pipe', 'pipe'] }
)

sp.stdout.pipe(process.stdout)
sp.stderr.pipe(process.stdout)
sp.stdin.write('this input will become output\n')
sp.stdin.end()

// Risultato
$ node example52.js
err output
this input will become output
```