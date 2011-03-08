## Extensions

Les extensions sont des objets executables partagés dynamiquement
(comme des .dll ou .so). Ils peuvent offrir la "glue" avec les bibliothèques
C et C++. L'API (pour le moment) est plutôt complexe et implique une
connaissance de plusieurs bibliothèques :

 - V8 JavaScript, une bibliothèque C++. Elle est utilisée pour l'interface
   avec JavaScript : créer des objets, appeler des fonctions, etc. Elle
   est principalement documentée dans son fichier d'en-tête `v8.h` 
   trouvable à `/deps/v8/include/v8.h` dans les sources de Node.

 - libev, une boucle d'évènements en C. Lorsqu'il y a besoin d'attendre
   qu'un fichier soit lisible, attendre un compteur ou attendre un signal
   il faut utiliser l'interface de libev. En bref, si vous faites
   des entrées/sorties, vous devez utiliser libev. Node utilise la boucle
   `EV_DEFAULT`. La documentation se trouve [ici](http://cvs.schmorp.de/libev/ev.html)

 - libeio, un reservoir de threads en C. Elle est utilisée pour que les appels
   POSIX bloquants puissent être utilisés de manière asynchrone. Une couche
   d'abstraction existe pour la plupart de ces appels dans `src/file.cc` ;
   vous n'aurez alors sans doute pas à l'utiliser. Si vous en avez vraiment
   besoin regardez le fichier d'en-tête `deps/libeio/eio.h`

 - Les bibliothèques internes de Node. La plus importante est la classe
   `node::ObjectWrap` de laquelle vous pouvez faire hériter vos classes

 - Autres. Regardez dans `deps/` pour voir ce qui est disponible

Node compile statiquement ses dépendences dans l'executable. Lorsque vous
compilez votre extension, vous n'avez pas à vous préoccuper de lier ces
bibliothèques.

Pour se mettre en jambes, faisons une petite extensions qui fait ceci mais
en C++ : 

    exports.coucou = 'salut';

Pour commencer, nous créons un fichier `coucou.cc` :

    #include <v8.h>

    using namespace v8;

    extern "C" void
    init (Handle<Object> target)
    {
      HandleScope scope;
      target->Set(String::New("coucou"), String::New("salut"));
    }

Ce code source doit être compilé en un fichier d'extension binaire
`coucou.node`. Pour ce faire, nous devons créer un fichier `wscript` qui
est du code python et ressemble à ceci :

    srcdir = '.'
    blddir = 'build'
    VERSION = '0.0.1'

    def set_options(opt):
      opt.tool_options('compiler_cxx')

    def configure(conf):
      conf.check_tool('compiler_cxx')
      conf.check_tool('node_addon')

    def build(bld):
      obj = bld.new_task_gen('cxx', 'shlib', 'node_addon')
      obj.target = 'coucou'
      obj.source = 'coucou.cc'

La commande `node-waf configure build` crée le fichier
`build/default/coucou.node` qui est notre extension

`node-waf` n'est que [WAF](http://code.google.com/p/waf), le système de
compilation en python. `node-waf` est disponible pour la facilité 
d'utilisation


Toutes les extensions doivent exporter une fonction nommée `init` avec
ce prototype :

    extern 'C' void init (Handle<Object> target)

Pour l'instant, ceci est la seule documentation sur les extensions. Voir
<http://github.com/joyent/node_postgres> pour un exemple réel.
