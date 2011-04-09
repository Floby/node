## Child Processes

Node expose un `popen(3)` tridirectionnel à travers la classe `ChildProcess`

Il est possible de manipuler les flux `stdin`, `stdout` et `stderr` du
processus enfant d'une manière complètement non-bloquante

Pour créer un processus fils, il faut utiliser `require('child_process').spawn()`

Les processus fils ont toujours trois flux qui leur sont associés. `child.stdin`,
`child.stdout` et `child.stderr`.

`ChildProcess` est un emetteur de la classe `EventEmitter`

### Evènement:  'exit'

`function (code, signal) {}`

Cet évènement est émis lorsque le processus fils termine. Si le processus s'est
terminé normalement, `code` est le code de retour du processus, sinon `null`.
Si le processus s'est terminé suite à la réception d'un signal, `signal` est le 
nom de ce signal, sinon `null`

voir `waitpid(2)`.

### child.stdin

Un flux en écriture qui représente l'entrée standard `stdin` du processus fils.
Fermer ce flux avec `end()` provoque souvent la temrinaison du processus fils.

### child.stdout

Un flux en lecture qui représente la sortie standard `stdout` du processus fils.

### child.stderr

Un flux en lecture qui représente la sortie d'erreurs `stderr` du processus fils.

### child.pid

L'identifiant du processus fils.

Exemple:

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Lancé processus fils : ' + grep.pid);
    grep.stdin.end();


### child_process.spawn(commande, args=[], [options])

Lance un nouveau processus avec la `commande` donnée et `args` comme arguments.
Si omis, `args` est égal à un tableau vide.

Le troisième argument est utilisé pour des options avancées, dont les valeurs
par défaut sont :

    { cwd: undefined,
      env: process.env,
      customFds: [-1, -1, -1],
      setsid: false
    }

`cwd` permet de spécifier le repertoire de travail dans lequel le processus sera lancé.
`env` permet de spécifier des variables d'environnement additionnelle pour le processus fils.
Avec `customFds`, il est possible de 'brancher' les entrées et sorties standard du  processus
fils sur des flux existants. -1 signifie qu'un nouveau flux doit être créé.
`setsid`, si `true` permet de lancer le processus fils dans une nouvelle session.

Exemple : lancer `ls -lh /usr` en récupérant `stdout`, `stderr` et le code de retour

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
    });

    ls.on('exit', function (code) {
      console.log("le processus fils s'est terminé avec le code " + code);
    });


Exemple : une version capillotractée de `ps ax | grep ssh`

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      console.log('ps stderr: ' + data);
    });

    ps.on('exit', function (code) {
      if (code !== 0) {
        console.log("ps s'est terminé avec le code " + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      console.log(data);
    });

    grep.stderr.on('data', function (data) {
      console.log('grep stderr: ' + data);
    });

    grep.on('exit', function (code) {
      if (code !== 0) {
        console.log("grep s'est terminé avec le code " + code);
      }
    });


Exemple : verifier si un processus s'est terminé correctement

    var spawn = require('child_process').spawn,
        child = spawn('mauvaise_commande');

    child.stderr.setEncoding('utf8');
    child.stderr.on('data', function (data) {
      if (/^execvp\(\)/.test(data)) {
        console.log('Impossible de démarrer le processus fils');
      }
    });


voir aussi : `child_process.exec()`

### child_process.exec(commande, [options], callback)

Fonction de haut niveau permettant d'éxecuter une commande par un
processus fils, mettre les sorties en tampon mémoire et d'appeler
une fonction de rappel.

    var util = require('util'),
        exec = require('child_process').exec,
        child;

    child = exec('cat *.js mauvais_fichier | wc -l',
      function (erreur, stdout, stderr) {
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
        if (erreur !== null) {
          console.log('erreur avec exec : ' + erreur);
        }
    });

La fonction de rappel reçoit les arguments `(erreur, stdout, stderr)`. en cas de succès,
`erreur` est `null`. En cas d'échec, `erreur` est de la classe `Error` et `erreur.code`
contient le code de retour du processus fils ; `erreur.signal` est le signal qui a causé
la terminaison du processus.

Le second argument est optionnel et permet de spécifier plusieurs options dont les valeurs
par défaut sont :

    { encoding: 'utf8',
      timeout: 0,
      maxBuffer: 200*1024,
      killSignal: 'SIGTERM',
      cwd: null,
      env: null }

Si `timeout` est plus grand que 0, alors le processus fils sera tué s'il met plus
que `timeout` millisecondes à s'éxecuter. Le processus fils se voit alors envoyer le signal
`killSignal`.
`maxBuffer` est la plus grande quantité de données pouvant être mise en tampon pour `stderr` et `stdout`.
Si cette valeur est dépassée, le processus fils sera tué.


### child.kill(signal='SIGTERM')

Envoie un signal au processus fils. Sans arguement, le signal `'SIGTERM'` est envoyé.
voir `singal(7)` pour une liste détaillées des signaux.

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('exit', function (code, signal) {
      console.log('processus fils terminé après reception du signal '+signal);
    });

    // envoi du signal SIGHUP
    grep.kill('SIGHUP');

Il est à noter que la fonction `kill` (tuer en anglais) ne tue pas nécéssairement
le processus fils. Elle envoie seulement un signal au processus.

voir `kill(2)`
