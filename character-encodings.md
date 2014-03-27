Let's start with a couple of definitions:

A **character** is what a human would write in one or multiple strikes. Example characters are a, Ä, 中, €, ☺. A character string is, well, a string of characters, like "aÄ中€☺" .

**Unicode** is not an encoding, but a list that strives to include all possible characters. For example, a, Ä, 中, €, and ☺ occupy the places (we call those places codepoints) 97, 196, 20013, 8364, 9786 in that list. Similar characters are often grouped together in the list for convenience. For example, character 98 is b, character 99 c, and so on.

While humans can work with characters intuitively (and could work with list of codepoints), computers need an encoding into bits and bytes. Let's assume these are always 8 bit long, so they can express 256 different values, commonly rendered as a decimal number between 000 and 255, or a hexadecimal number between 00 and ff, or a binary number between 0000 0000 and 1111 1111.

Unicode contains less than 2^32 numbers, so one could simply encode each codepoint into 4 bytes, just like a regular integer, like this:

    a (   97) -> 00 00 00 61
    Ä (  196) -> 00 00 00 c4
    中 (20013) -> 00 00 4e 2d
    ☺ ( 9786) -> 00 00 26 3a
    € ( 8364) -> 00 00 20 ac

This encoding is called UTF-32-BE (=Big Endian. There's also the Little Endian variant, which turns around the bytes, i.e. encodes € as  ac 20 00 00 . See http://en.wikipedia.org/wiki/Endianess for details ). As you can see, it's pretty wasteful - for virtually all characters in this email, the first three bytes are zero!

Some encodings deal with this by simply throwing an error for some characters their inventors consider unnecessary. For example, latin1 would encode the above as:

    a (   97) -> 61
    Ä (  196) -> c4
    中 (20013) -> Error
    ☺ ( 9786) -> Error
    € ( 8364) -> Error

Needless to say, these encodings are stupid (despite being efficient) and should not be used in sane software. To top it off, there are a lot of different of these stupid encodings, each picking a different subset of characters. For example, here is CP1252, the default in Western European Windows:

    a (   97) -> 61
    Ä (  196) -> c4
    中 (20013) -> Error
    ☺ ( 9786) -> Error
    € ( 8364) -> 80

In fact, ASCII was and is widely used in the US, and contains only 127 characters:

    a (   97) -> 61
    Ä (  196) -> Error
    中 (20013) -> Error
    ☺ ( 9786) -> Error
    € ( 8364) -> Error

However, there is a tremendous incentive to maintain backwards compatibility and keep using these old encodings. Imagine that the next version Windows would ship with a different encoding. Lots of software would suddenly break upon reading files produced with the old versions. Some software bugs may be uncovered as well, but the end result would be catastrophic for Microsoft; all news outlets would report that the new version of Windows breaks all software and causes catastrophic errors, and nobody would buy it.

Of course, software can carry the correct encoding in metadata or add a special marker at the start of the string, but since many programmers don't understand encodings, they typically ignore both.

Fortunately, there is a neat encoding, UTF-8, which is backwards-compatible with ASCII, produces a small output for Western (and many other commonly used) characters, *and* can encode ALL characters:

    a (   97) -> 61
    Ä (  196) -> c3 84
    中 (20013) -> e4 b8 ad
    ☺ ( 9786) -> e2 82 ac
    € ( 8364) -> e2 89 ba

UTF-8 does this by storing the number of bytes in the first byte, like this (x are the bits of the codepoint in binary):

    Codepoint < 2^7 (between 0 and 127):
    0xxxxxxx          
    Codepoint < 2^11 (between 128 and 2047):
    110xxxxx 10xxxxxx
    Codepoint < 2^16 (between 2048 and 65535):
    1110xxxxx 10xxxxxx 10xxxxxx
    Codepoint < 2^21 (between 65536 and 2097151):
    1110xxxxx 10xxxxxx 10xxxxxx 10xxxxxx

and so on. Since the encoder must pick the shortest possible form for each codepoint, there are also a couple of invalid sequences, such as 11000000 10000001, which could be written shorter as 00000001 . On the other hand, that means that there is only one possible encoding for each character, and that's often a good thing.

Good text-handling software uses either only UTF-8, or UTF-8 by default. For example, this includes all modern Web browsers, and many programs on Linux and Mac OS X.
