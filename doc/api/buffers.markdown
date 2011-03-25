## Tampons mémoire

Le JavaScript pur est très bon avec l'Unicode mais peine avec les données
binaires. Quand on traîte des flux TCP ou le système de fichiers, il est
nécéssaire de gérer les flux d'octets. Node dispose de plusieurs méthodes
pour manipuler, créer et lire des flux binaires.

Les données binaires sont stockées dans des instance de la classe `Buffer`.
Un `buffer` est similaire à un tableau d'entiers mais correspond aux données
binaires allouée en dehors de la mémoire gérée par V8. On ne peut pas 
changer la taille d'un `buffer`.

Le constructeur `Buffer` est global.

Convertir des tampons vers des chaînes de caractères JavaScript et vice 
versa demande un encodage/décodage explicite. Les encodages sont les 
suivants : 

* `'ascii'` - pour les données ASCII à 7 bits seulements. Cette méthode
d'encodage est très rapide et enlèvera le bit fort si présent.

* `'utf8'` - Caractères Unicode. Beaucoup de pages web et d'autre formats de documents utilisent UTF-8.

* `'ucs2'` - 2-bytes, little endian encoded Unicode characters. It can encode
only BMP(Basic Multilingual Plane, U+0000 - U+FFFF).
* `'usc2'` - Caractères sur deux octets petit-boutistes (little endian). Ne peux décoder
que les BMP(Basic Multilingual Plane, de U+0000 à U+FFFF)

* `'base64'` - encodage en Base64.

* `'binary'` - Une méthode d'encodage spécifique à Node n'utilisant que les
8 premiers bits de chaque caractère. Cette méthode est dépréciée et devrait
être évitée en faveur d'objets `Buffer`. Elle sera retirée des prochaines
versions de Node.

* `'hex'` - Encode chaque octet en deux caractères hexadécimaux.


### new Buffer(size)

Alloue un nouveau `buffer` de taille `size` (en octets).

### new Buffer(array)

Alloue un nouveau `buffer` en utilisant un tableau d'octets.

### new Buffer(str, encoding='utf8')

Alloue un nouveau `buffer` contenant la chaîne de caractère donnée par `str`.

### buffer.write(string, offset=0, encoding='utf8')

Writes `string` to the buffer at `offset` using the given encoding. Returns
number of octets written.  If `buffer` did not contain enough space to fit
the entire string, it will write a partial amount of the string. In the case
of `'utf8'` encoding, the method will not write partial characters.

Example: write a utf8 string into a buffer, then print it

    buf = new Buffer(256);
    len = buf.write('\u00bd + \u00bc = \u00be', 0);
    console.log(len + " bytes: " + buf.toString('utf8', 0, len));

    // 12 bytes: ½ + ¼ = ¾


### buffer.toString(encoding, start=0, end=buffer.length)

Decodes and returns a string from buffer data encoded with `encoding`
beginning at `start` and ending at `end`.

See `buffer.write()` example, above.


### buffer[index]

Get and set the octet at `index`. The values refer to individual bytes,
so the legal range is between `0x00` and `0xFF` hex or `0` and `255`.

Example: copy an ASCII string into a buffer, one byte at a time:

    str = "node.js";
    buf = new Buffer(str.length);

    for (var i = 0; i < str.length ; i++) {
      buf[i] = str.charCodeAt(i);
    }

    console.log(buf);

    // node.js

### Buffer.isBuffer(obj)

Tests if `obj` is a `Buffer`.

### Buffer.byteLength(string, encoding='utf8')

Gives the actual byte length of a string.  This is not the same as
`String.prototype.length` since that returns the number of *characters* in a
string.

Example:

    str = '\u00bd + \u00bc = \u00be';

    console.log(str + ": " + str.length + " characters, " +
      Buffer.byteLength(str, 'utf8') + " bytes");

    // ½ + ¼ = ¾: 9 characters, 12 bytes


### buffer.length

The size of the buffer in bytes.  Note that this is not necessarily the size
of the contents. `length` refers to the amount of memory allocated for the
buffer object.  It does not change when the contents of the buffer are changed.

    buf = new Buffer(1234);

    console.log(buf.length);
    buf.write("some string", "ascii", 0);
    console.log(buf.length);

    // 1234
    // 1234

### buffer.copy(targetBuffer, targetStart=0, sourceStart=0, sourceEnd=buffer.length)

Does a memcpy() between buffers.

Example: build two Buffers, then copy `buf1` from byte 16 through byte 19
into `buf2`, starting at the 8th byte in `buf2`.

    buf1 = new Buffer(26);
    buf2 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
      buf2[i] = 33; // ASCII !
    }

    buf1.copy(buf2, 8, 16, 20);
    console.log(buf2.toString('ascii', 0, 25));

    // !!!!!!!!qrst!!!!!!!!!!!!!


### buffer.slice(start, end=buffer.length)

Returns a new buffer which references the
same memory as the old, but offset and cropped by the `start` and `end`
indexes.

**Modifying the new buffer slice will modify memory in the original buffer!**

Example: build a Buffer with the ASCII alphabet, take a slice, then modify one byte
from the original Buffer.

    var buf1 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
    }

    var buf2 = buf1.slice(0, 3);
    console.log(buf2.toString('ascii', 0, buf2.length));
    buf1[0] = 33;
    console.log(buf2.toString('ascii', 0, buf2.length));

    // abc
    // !bc
