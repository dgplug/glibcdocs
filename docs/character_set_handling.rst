6 Character Set Handling
========================

Character sets used in the early days of computing had only six, seven, or eight bits for
each character: there was never a case where more than eight bits (one byte) were used
to represent a single character. The limitations of this approach became more apparent
as more people grappled with non-Roman character sets, where not all the characters that
make up a language’s character set can be represented by 2 8 choices. This chapter shows
the functionality that was added to the C library to support multiple character sets.

6.1 Introduction to Extended Characters
---------------------------------------

A variety of solutions are available to overcome the differences between character sets with
a 1:1 relation between bytes and characters and character sets with ratios of 2:1 or 4:1.
The remainder of this section gives a few examples to help understand the design decisions
made while developing the functionality of the C library.

A distinction we have to make right away is between internal and external representation.
Internal representation means the representation used by a program while keeping the text
in memory. External representations are used when text is stored or transmitted through
some communication channel. Examples of external representations include files waiting in
a directory to be read and parsed.

Traditionally there has been no difference between the two representations. It was equally
comfortable and useful to use the same single-byte representation internally and externally.
This comfort level decreases with more and larger character sets.

One of the problems to overcome with the internal representation is handling text that
is externally encoded using different character sets. Assume a program that reads two texts
and compares them using some metric. The comparison can be usefully done only if the texts
are internally kept in a common format.

For such a common format (= character set) eight bits are certainly no longer enough.
So the smallest entity will have to grow: wide characters will now be used. Instead of one
byte per character, two or four will be used instead. (Three are not good to address in
memory and more than four bytes seem not to be necessary).

As shown in some other part of this manual, a completely new family has been created
of functions that can handle wide character texts in memory. The most commonly used
character sets for such internal wide character representations are Unicode and ISO 10646
(also known as UCS for Universal Character Set). Unicode was originally planned as a 16-
bit character set; whereas, ISO 10646 was designed to be a 31-bit large code space. The two
standards are practically identical. They have the same character repertoire and code table,
but Unicode specifies added semantics. At the moment, only characters in the first 0x10000
code positions (the so-called Basic Multilingual Plane, BMP) have been assigned, but the
assignment of more specialized characters outside this 16-bit space is already in progress.
A number of encodings have been defined for Unicode and ISO 10646 characters: UCS-2
is a 16-bit word that can only represent characters from the BMP, UCS-4 is a 32-bit word
than can represent any Unicode and ISO 10646 character, UTF-8 is an ASCII compatible
encoding where ASCII characters are represented by ASCII bytes and non-ASCII characters
by sequences of 2-6 non-ASCII bytes, and finally UTF-16 is an extension of UCS-2 in which
pairs of certain UCS-2 words can be used to encode non-BMP characters up to **0x10ffff**.

To represent wide characters the char type is not suitable. For this reason the ISO C
standard introduces a new type that is designed to keep one character of a wide character
string. To maintain the similarity there is also a type corresponding to int for those
functions that take a single wide character.

:wchar_t: This data type is used as the base type for wide character strings. In other 
          words, arrays of objects of this type are the equivalent of **char[]** for multibyte 
          character strings. The type is defined in **stddef.h**. 
          The ISO C90 standard, where wchar_t was introduced, does not say anything specific
          about the representation. It only requires that this type is capable of storing all
          elements of the basic character set. Therefore it would be legitimate to define wchar_t 
          as char, which might make sense for embedded systems.
          But in the GNU C Library **wchar_t** is always 32 bits wide and, therefore, capable of
          representing all UCS-4 values and, therefore, covering all of ISO 10646. Some Unix
          systems define wchar_t as a 16-bit type and thereby follow Unicode very strictly.
          This definition is perfectly fine with the standard, but it also means that to repre-
          sent all characters from Unicode and ISO 10646 one has to use UTF-16 surrogate
          characters, which is in fact a multi-wide-character encoding. But resorting to multi-
          wide-character encoding contradicts the purpose of the **wchar_t** type.

:wint_t: wint_t is a data type used for parameters and variables that contain a single wide
          character. As the name suggests this type is the equivalent of int when using the
          normal char strings. The types **wchar_t** and **wint_t** often have the same represen-
          tation if their size is 32 bits wide but if wchar_t is defined as char the type **wint_t**
          must be defined as int due to the parameter promotion.
          This type is defined in wchar.h and was introduced in Amendment 1 to ISO C90.

As there are for the char data type macros are available for specifying the minimum
and maximum value representable in an object of type **wchar_t**.

:wint_t WCHAR_MIN: The macro **WCHAR_MIN** evaluates to the minimum value representable by an object of
                   type **wint_t**.
                   This macro was introduced in Amendment 1 to ISO C90.

:wint_t WCHAR_MAX: The macro **WCHAR_MAX** evaluates to the maximum value representable by an object of
                   type **wint_t**.
                   This macro was introduced in Amendment 1 to ISO C90.

Another special wide character value is the equivalent to EOF.

:wint_t WEOF: The macro WEOF evaluates to a constant expression of type **wint_t** whose value is
              different from any member of the extended character set.

WEOF need not be the same value as EOF and unlike EOF it also need not be negative.
In other words, sloppy code like


::

    {
        int c;
        ...
        while ((c = getc (fp)) < 0)
        ...
    }

has to be rewritten to use WEOF explicitly when wide characters are used:


::

    {
        wint_t c;
        ...
        while ((c = wgetc (fp)) != WEOF)
        ...
    }

This macro was introduced in Amendment 1 to ISO C90 and is defined in **wchar.h**.


These internal representations present problems when it comes to storage and transmit-
tal. Because each single wide character consists of more than one byte, they are affected by
byte-ordering. Thus, machines with different endianesses would see different values when
accessing the same data. This byte ordering concern also applies for communication pro-
tocols that are all byte-based and therefore require that the sender has to decide about
splitting the wide character in bytes. A last (but not least important) point is that wide
characters often require more storage space than a customized byte-oriented character set.

For all the above reasons, an external encoding that is different from the internal encoding
is often used if the latter is UCS-2 or UCS-4. The external encoding is byte-based and can
be chosen appropriately for the environment and for the texts to be handled. A variety of
different character sets can be used for this external encoding (information that will not
be exhaustively presented here–instead, a description of the major groups will suffice). All
of the ASCII-based character sets fulfill one requirement: they are "filesystem safe." This
means that the character '/' is used in the encoding only to represent itself. Things are a
bit different for character sets like EBCDIC (Extended Binary Coded Decimal Interchange
Code, a character set family used by IBM), but if the operating system does not understand
EBCDIC directly the parameters-to-system calls have to be converted first anyhow.

+ The simplest character sets are single-byte character sets. There can be only up to
  256 characters (for 8 bit character sets), which is not sufficient to cover all languages
  but might be sufficient to handle a specific text. Handling of a 8 bit character sets is
  simple. This is not true for other kinds presented later, and therefore, the application
  one uses might require the use of 8 bit character sets.
+ The ISO 2022 standard defines a mechanism for extended character sets where one
  character can be represented by more than one byte. This is achieved by associating a
  state with the text. Characters that can be used to change the state can be embedded
  in the text. Each byte in the text might have a different interpretation in each state.
  The state might even influence whether a given byte stands for a character on its own
  or whether it has to be combined with some more bytes.
+ In most uses of ISO 2022 the defined character sets do not allow state changes that
  cover more than the next character. This has the big advantage that whenever one
  can identify the beginning of the byte sequence of a character one can interpret a text
  correctly. Examples of character sets using this policy are the various EUC character
  sets (used by Sun’s operating systems, EUC-JP, EUC-KR, EUC-TW, and EUC-CN)
  or Shift JIS (SJIS, a Japanese encoding).
  
  But there are also character sets using a state that is valid for more than one character
  and has to be changed by another byte sequence. Examples for this are ISO-2022-JP,
  ISO-2022-KR, and ISO-2022-CN.

+ Early attempts to fix 8 bit character sets for other languages using the Roman alphabet
  lead to character sets like ISO 6937. Here bytes representing characters like the acute
  accent do not produce output themselves: one has to combine them with other charac-
  ters to get the desired result. For example, the byte sequence 0xc2 0x61 (non-spacing
  acute accent, followed by lower-case ‘a’) to get the “small a with acute” character. To
  get the acute accent character on its own, one has to write 0xc2 0x20 (the non-spacing
  acute followed by a space).
  Character sets like ISO 6937 are used in some embedded systems such as teletex.

+ Instead of converting the Unicode or ISO 10646 text used internally, it is often also
  sufficient to simply use an encoding different than UCS-2/UCS-4. The Unicode and
  ISO 10646 standards even specify such an encoding: UTF-8. This encoding is able to
  represent all of ISO 10646 31 bits in a byte string of length one to six.
  
  There were a few other attempts to encode ISO 10646 such as UTF-7, but UTF-8 is
  today the only encoding that should be used. In fact, with any luck UTF-8 will soon be
  the only external encoding that has to be supported. It proves to be universally usable
  and its only disadvantage is that it favors Roman languages by making the byte string
  representation of other scripts (Cyrillic, Greek, Asian scripts) longer than necessary if
  using a specific character set for these scripts. Methods like the Unicode compression
  scheme can alleviate these problems.

The question remaining is: how to select the character set or encoding to use. The
answer: you cannot decide about it yourself, it is decided by the developers of the system
or the majority of the users. Since the goal is interoperability one has to use whatever the
other people one works with use. If there are no constraints, the selection is based on the
requirements the expected circle of users will have. In other words, if a project is expected
to be used in only, say, Russia it is fine to use KOI8-R or a similar character set. But if
at the same time people from, say, Greece are participating one should use a character set
that allows all people to collaborate.

The most widely useful solution seems to be: go with the most general character set,
namely ISO 10646. Use UTF-8 as the external encoding and problems about users not
being able to use their own language adequately are a thing of the past.

One final comment about the choice of the wide character representation is necessary
at this point. We have said above that the natural choice is using Unicode or ISO 10646.
This is not required, but at least encouraged, by the ISO C standard. The standard defines
at least a macro **__STDC_ISO_10646__** that is only defined on systems where the wchar_t
type encodes ISO 10646 characters. If this symbol is not defined one should avoid making
assumptions about the wide character representation. If the programmer uses only the
functions provided by the C library to handle wide character strings there should be no
compatibility problems with other systems.

6.2 Overview about Character Handling Functions
-----------------------------------------------

A Unix C library contains three different sets of functions in two families to handle character
set conversion. One of the function families (the most commonly used) is specified in the
ISO C90 standard and, therefore, is portable even beyond the Unix world. Unfortunately
this family is the least useful one. These functions should be avoided whenever possible,
especially when developing libraries (as opposed to applications).

The second family of functions got introduced in the early Unix standards (XPG2) and
is still part of the latest and greatest Unix standard: Unix 98. It is also the most powerful
and useful set of functions. But we will start with the functions defined in Amendment 1
to ISO C90.


6.3 Restartable Multibyte Conversion Functions
----------------------------------------------

The ISO C standard defines functions to convert strings from a multibyte representation to
wide character strings. There are a number of peculiarities:

+ The character set assumed for the multibyte encoding is not specified as an argument
  to the functions. Instead the character set specified by the LC_CTYPE category of the
  current locale is used; see Section 7.3 [Locale Categories], page 170.

+ The functions handling more than one character at a time require NUL terminated
  strings as the argument (i.e., converting blocks of text does not work unless one can
  add a NUL byte at an appropriate place). The GNU C Library contains some extensions
  to the standard that allow specifying a size, but basically they also expect terminated
  strings.

Despite these limitations the ISO C functions can be used in many contexts. In graphical
user interfaces, for instance, it is not uncommon to have functions that require text to be
displayed in a wide character string if the text is not simple ASCII. The text itself might
come from a file with translations and the user should decide about the current locale,
which determines the translation and therefore also the external encoding used. In such a
situation (and many others) the functions described here are perfect. If more freedom while
performing the conversion is necessary take a look at the iconv functions (see Section 6.5
[Generic Charset Conversion], page 148).

6.3.1 Selecting the conversion and its properties
#################################################

We already said above that the currently selected locale for the LC_CTYPE category decides
the conversion that is performed by the functions we are about to describe. Each locale
uses its own character set (given as an argument to localedef) and this is the one assumed
as the external multibyte encoding. The wide character set is always UCS-4 in the GNU C
Library.

A characteristic of each multibyte character set is the maximum number of bytes that
can be necessary to represent one character. This information is quite important when
writing code that uses the conversion functions (as shown in the examples below). The
ISO C standard defines two macros that provide this information.

:int MB_LEN_MAX: **MB_LEN_MAX** specifies the maximum number of bytes in the multibyte sequence for a single character in any of the supported locales. It is a compile-time constant and is defined in **limits.h**.

:int MB_CUR_MAX: **MB_CUR_MAX** expands into a positive integer expression that is the maximum number of bytes in a multibyte character in the current locale. The value is never greater than **MB_LEN_MAX**. Unlike **MB_LEN_MAX** this macro need not be a compile-time constant, and in the GNU C Library it is not.
                 **MB_CUR_MAX** is defined in stdlib.h.

Two different macros are necessary since strictly ISO C90 compilers do not allow variable
length array definitions, but still it is desirable to avoid dynamic allocation. This incomplete
piece of code shows the problem:


::

    {
        char buf[MB_LEN_MAX];
        ssize_t len = 0;
        while (! feof (fp))
        {
            fread (&buf[len], 1, MB_CUR_MAX - len, fp);
            /* . . . process buf */
            len -= used;
        }
    }


The code in the inner loop is expected to have always enough bytes in the array buf
to convert one multibyte character. The array buf has to be sized statically since many
compilers do not allow a variable size. The fread call makes sure that MB_CUR_MAX bytes
are always available in buf. Note that it isn’t a problem if MB_CUR_MAX is not a compile-time
constant.


6.3.2 Representing the state of the conversion
##############################################

In the introduction of this chapter it was said that certain character sets use a stateful
encoding. That is, the encoded values depend in some way on the previous bytes in the
text.
Since the conversion functions allow converting a text in more than one step we must
have a way to pass this information from one call of the functions to another.

:mbstate_t: A variable of type mbstate_t can contain all the information about the shift state
            needed from one call to a conversion function to another.
            **mbstate_t** is defined in **wchar.h**. It was introduced in Amendment 1 to ISO C90.

To use objects of type mbstate_t the programmer has to define such objects (normally
as local variables on the stack) and pass a pointer to the object to the conversion functions.
This way the conversion function can update the object if the current multibyte character
set is stateful.

There is no specific function or initializer to put the state object in any specific state.
The rules are that the object should always represent the initial state before the first use,
and this is achieved by clearing the whole variable with code such as follows:


::

    {
        mbstate_t state;
        memset (&state, '\0', sizeof (state));
        /* from now on state can be used. */
        ...
    }

When using the conversion functions to generate output it is often necessary to test
whether the current state corresponds to the initial state. This is necessary, for example,
to decide whether to emit escape sequences to set the state to the initial state at certain
sequence points. Communication protocols often require this.

:int mbsinit ( const mbstate t * ps ): Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety Concepts], page 2.
                                      The **mbsinit** function determines whether the state object pointed to by ps is in the initial state. If ps is a null pointer or the object is in the initial state the return value is nonzero. Otherwise it is zero.
                                      **mbsinit** was introduced in Amendment 1 to ISO C90 and is declared in **wchar.h**.

Code using mbsinit often looks similar to this:


::


    {
        mbstate_t state;
        memset (&state, '\0', sizeof (state));
        /* Use state. */
        ...
        if (! mbsinit (&state))
        {
            /* Emit code to return to initial state. */
            const wchar_t empty[] = L"";
            const wchar_t *srcp = empty;
            wcsrtombs (outbuf, &srcp, outbuflen, &state);
        }
        ...
    }

The code to emit the escape sequence to get back to the initial state is interesting. The
wcsrtombs function can be used to determine the necessary output code (see Section 6.3.4
[Converting Multibyte and Wide Character Strings], page 139). Please note that with the
GNU C Library it is not necessary to perform this extra action for the conversion from
multibyte text to wide character text since the wide character encoding is not stateful. But
there is nothing mentioned in any standard that prohibits making wchar_t use a stateful
encoding.

6.3.3 Converting Single Characters
##################################

The most fundamental of the conversion functions are those dealing with single characters.
Please note that this does not always mean single bytes. But since there is very often
a subset of the multibyte character set that consists of single byte sequences, there are
functions to help with converting bytes. Frequently, ASCII is a subset of the multibyte
character set. In such a scenario, each ASCII character stands for itself, and all other
characters have at least a first byte that is beyond the range 0 to 127.

:wint_t btowc ( int c ): Preliminary: | MT-Safe | AS-Unsafe corrupt heap lock dlopen | AC-Unsafe corrupt lock mem fd | See Section 1.2.2.1 [POSIX Safety Concepts], page 2. The btowc function (“byte to wide character”) converts a valid single byte character c in the initial shift state into the wide character equivalent using the conversion rules from the currently selected locale of the **LC_CTYPE** category.

                         If **(unsigned char)** c is no valid single byte multibyte character or if c is **EOF**, the function returns **WEOF**.

Please note the restriction of c being tested for validity only in the initial shift state.
No mbstate_t object is used from which the state information is taken, and the
function also does not use any static state.
The btowc function was introduced in Amendment 1 to ISO C90 and is declared in
**wchar.h**.

Despite the limitation that the single byte value is always interpreted in the initial state,
this function is actually useful most of the time. Most characters are either entirely single-
byte character sets or they are extensions to ASCII. But then it is possible to write code
like this (not that this specific example is very useful):


::

    wchar_t *
    itow (unsigned long int val)
    {
        static wchar_t buf[30];
        wchar_t *wcp = &buf[29];
        *wcp = L'\0';
        while (val != 0)
        {
            *--wcp = btowc ('0' + val % 10);
            val /= 10;
        }
        if (wcp == &buf[29])
        *--wcp = L'0';
        return wcp;
    }

Why is it necessary to use such a complicated implementation and not simply cast '0'
+ val % 10 to a wide character? The answer is that there is no guarantee that one can
perform this kind of arithmetic on the character of the character set used for wchar_t
representation. In other situations the bytes are not constant at compile time and so the
compiler cannot do the work. In situations like this, using btowc is required.
There is also a function for the conversion in the other direction.

