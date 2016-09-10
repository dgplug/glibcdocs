================
Pattern Matching
================

The GNU C Library provides pattern matching facilities for two kinds of patterns: regular
expressions and file-name wildcards. The library also provides a facility for expanding
variable and command references and parsing text into words in the way the shell does.


Wildcard Matching
=================

This section describes how to match a wildcard pattern against a particular string. The
result is a yes or no answer: does the string fit the pattern or not. The symbols described
here are all declared in ``fnmatch.h``.

``int fnmatch ( const char *pattern, const char *string, int flags )``        [Function]

    Preliminary:  | MT-Safe env locale | AS-Unsafe heap | AC-Unsafe mem |
                  See Section [POSIX Safety Concepts].

    This function tests whether the string string matches the pattern pattern. It re-
    turns 0 if they do match; otherwise, it returns the nonzero value ``FNM_NOMATCH``. The
    arguments ``pattern`` and ``string`` are both strings.
    The argument flags is a combination of flag bits that alter the details of matching.
    See below for a list of the defined flags.
    In the GNU C Library, ``fnmatch`` might sometimes report “errors” by returning nonzero
    values that are not equal to ``FNM_NOMATCH``.

    These are the available flags for the ``flags`` argument:

``FNM_FILE_NAME``

    Treat the ``'/'`` character specially, for matching file names. If this flag is set,
    wildcard constructs in ``pattern`` cannot match ``'/'`` in ``string``. Thus, the only way
    to match ``'/'`` is with an explicit ``'/'`` in ``pattern``.


``FNM_PATHNAME``

    This is an alias for ``FNM_FILE_NAME``; it comes from POSIX.2. We don’t recom-
    mend this name because we don’t use the term ``"pathname"`` for file names.

``FNM_PERIOD``

    Treat the ``'.'`` character specially if it appears at the beginning of ``string``. If this
    flag is set, wildcard constructs in ``pattern`` cannot match ``.`` as the first character
    of ``string``.

    If you set both FNM_PERIOD and FNM_FILE_NAME, then the special treatment
    applies to ``'.'`` following ``'/'`` as well as to ``'.'`` at the beginning of ``string``. (The
    shell uses the ``FNM_PERIOD`` and ``FNM_FILE_NAME`` flags together for matching file
    names.)

``FNM_NOESCAPE``

    Don't treat the ``'\'`` character specially in patterns. Normally, ``'\'`` quotes the
    following character, turning off its special meaning (if any) so that it matches
    only itself. When quoting is enabled, the pattern ``'\?'`` matches only the string
    ``'?'``, because the question mark in the pattern acts like an ordinary character.

    If you use ``FNM_NOESCAPE``, then ``'\'`` is an ordinary character.

``FNM_LEADING_DIR``

    Ignore a trailing sequence of characters starting with a ``'/'`` in ``string``; that is to
    say, test whether ``string`` starts with a directory name that ``pattern`` matches.
    If this flag is set, either ``'foo*'`` or ``'foobar'`` as a pattern would match the string
    ``'foobar/frobozz'``.

``FNM_CASEFOLD``

    Ignore case in comparing ``string`` to pattern.

``FNM_EXTMATCH``

    Besides the normal patterns, also recognize the extended patterns introduced
    in ``ksh``. The patterns are written in the form explained in the following table
    where ``pattern-list`` is a | separated list of patterns.

    ``?(pattern-list)``

        The pattern matches if zero or one occurrences of any of the pat-
        terns in the ``pattern-list`` allow matching the input string.

    ``*(pattern-list)``

        The pattern matches if zero or more occurrences of any of the pat-
        terns in the ``pattern-list`` allow matching the input string.

    ``+(pattern-list)``

        The pattern matches if one or more occurrences of any of the pat-
        terns in the ``pattern-list`` allow matching the input string.

    ``@(pattern-list)``

        The pattern matches if exactly one occurrence of any of the patterns
        in the ``pattern-list`` allows matching the input string.

    ``!(pattern-list)``

        The pattern matches if the input string cannot be matched with
        any of the patterns in the ``pattern-list``.

Globbing
========

The archetypal use of wildcards is for matching against the files in a directory, and making
a list of all the matches. This is called ``globbing``.

You could do this using ``fnmatch``, by reading the directory entries one by one and testing
each one with ``fnmatch``. But that would be slow (and complex, since you would have to
handle subdirectories by hand).

The library provides a function ``glob`` to make this particular use of wildcards convenient.
``glob`` and the other symbols in this section are declared in ``glob.h``.

Calling glob
------------

The result of globbing is a vector of file names (strings). To return this vector, ``glob`` uses a
special data type, ``glob_t``, which is a structure. You pass ``glob`` the address of the structure,
and it fills in the structure’s fields to tell you about the results.

``glob_t``                                            [Data Type]

    This data type holds a pointer to a word vector. More precisely, it records both the
    address of the word vector and its size. The GNU implementation contains some
    more fields which are non-standard extensions.

    ``gl_pathc``

        The number of elements in the vector, excluding the initial null entries if
        the ``GLOB DOOFFS`` flag is used (see ``gl offs`` below).

    ``gl_pathv``

        The address of the vector. This field has type ``char **``.

    ``gl_offs``

        The offset of the first real element of the vector, from its nominal address
        in the ``gl_pathv`` field. Unlike the other fields, this is always an input to
        ``glob``, rather than an output from it.

        If you use a nonzero offset, then that many elements at the beginning
        of the vector are left empty. (The glob function fills them with null
        pointers.)

        The ``gl_offs`` field is meaningful only if you use the ``GLOB_DOOFFS`` flag.
        Otherwise, the offset is always zero regardless of what is in this field, and
        the first real element comes at the beginning of the vector.

    ``gl_closedir``

        The address of an alternative implementation of the ``closedir`` function.
        It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter. The
        type of this field is ``void (*) (void *)``.
        This is a GNU extension.

    ``gl_readdir``

        The address of an alternative implementation of the ``readdir``
        function used to read the contents of a directory. It is used if the
        ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter. The type of this field
        is ``struct dirent *(*) (void *)``.

        An implementation of ``gl_readdir`` needs to initialize the following mem-
        bers of the ``struct dirent`` object:

        ``d_type``

            This member should be set to the file type of the entry if
            it is known. Otherwise, the value ``DT_UNKNOWN`` can be used.
            The ``glob`` function may use the specified file type to avoid
            callbacks in cases where the file type indicates that the data
            is not required.

        ``d_ino``

            This member needs to be non-zero, otherwise ``glob`` may skip
            the current entry and call the ``gl_readdir`` callback function
            again to retrieve another entry.

        ``d_name``

            This member must be set to the name of the entry. It must
            be null-terminated.

        The example below shows how to allocate a ``struct dirent`` object con-
        taining a given name.

        .. code-block:: c

            #include <dirent.h>
            #include <errno.h>
            #include <stddef.h>
            #include <stdlib.h>
            #include <string.h>

            struct dirent *
            mkdirent (const char *name)
            {
                size_t dirent_size = offsetof (struct dirent, d_name) + 1;
                size_t name_length = strlen (name);
                size_t total_size = dirent_size + name_length;
                if (total_size < dirent_size)
                {
                    errno = ENOMEM;
                    return NULL;
                }
                struct dirent *result = malloc (total_size);
                if (result == NULL)
                    return NULL;
                result->d_type = DT_UNKNOWN;
                result->d_ino = 1;      /* Do not skip this entry. */
                memcpy (result->d_name, name, name_length + 1);
                return result;
            }

        The ``glob`` function reads the ``struct dirent`` members listed above and
        makes a copy of the file name in the ``d_name member`` immediately after
        the ``gl_readdir`` callback function returns. Future invocations of any of
        the callback functions may dealloacte or reuse the buffer. It is the respon-
        sibility of the caller of the ``glob`` function to allocate and deallocate the
        buffer, around the call to ``glob`` or using the callback functions. For exam-
        ple, an application could allocate the buffer in the ``gl_readdir`` callback
        function, and deallocate it in the ``gl_closedir`` callback function.

        The ``gl_readdir`` member is a GNU extension.

    ``gl_opendir``

        The address of an alternative implementation of the ``opendir`` function.
        It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter. The
        type of this field is ``void *(*) (const char *)``.

        This is a GNU extension.

    ``gl_stat``

        The address of an alternative implementation of the ``stat`` function to get
        information about an object in the filesystem. It is used if the ``GLOB_
        ALTDIRFUNC`` bit is set in the flag parameter. The type of this field is
        ``int (*) (const char *, struct stat *)``.

        This is a GNU extension.

    ``gl_lstat``

        The address of an alternative implementation of the ``lstat`` function to
        get information about an object in the filesystems, not following symbolic
        links. It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter.
        The type of this field is ``int (*) (const char *, struct stat *)``.

        This is a GNU extension.

    ``gl_flags``

        The flags used when ``glob`` was called. In addition, ``GLOB_MAGCHAR`` might
        be set. See Section [Flags for Globbing] for more details.

        This is a GNU extension.


For use in the ``glob64`` function ``glob.h`` contains another definition for a very similar type.
``glob64_t`` differs from ``glob_t`` only in the types of the members ``gl_readdir``, ``gl_stat``, and
``gl_lstat``.


``globe64_t``                                           [Data Type]

    This data type holds a pointer to a word vector. More precisely, it records both the
    address of the word vector and its size. The GNU implementation contains some
    more fields which are non-standard extensions.

    ``gl_pathc``

        The number of elements in the vector, excluding the initial null entries if
        the ``GLOB_DOOFFS`` flag is used (see ``gl_offs`` below).

    ``gl_pathv``

        The address of the vector. This field has type ``char **``.

    ``gl_offs``

        The offset of the first real element of the vector, from its nominal address
        in the ``gl_pathv field``. Unlike the other fields, this is always an input to
        ``glob``, rather than an output from it.

        If you use a nonzero offset, then that many elements at the beginning
        of the vector are left empty. (The ``glob`` function fills them with null
        pointers.)

        The ``gl_offs`` field is meaningful only if you use the ``GLOB_DOOFFS`` flag.
        Otherwise, the offset is always zero regardless of what is in this field, and
        the first real element comes at the beginning of the vector.

    ``gl_closedir``

        The address of an alternative implementation of the ``closedir`` function.
        It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter. The
        type of this field is ``void (*) (void *)``.

        This is a GNU extension.

    ``gl_readdir``

        The address of an alternative implementation of the ``readdir64`` func-
        tion used to read the contents of a directory. It is used if the ``GLOB_
        ALTDIRFUNC`` bit is set in the flag parameter. The type of this field is
        ``struct dirent64 *(*) (void *)``.

        This is a GNU extension.

    ``gl_opendir``

        The address of an alternative implementation of the ``opendir`` function.
        It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter. The
        type of this field is ``void *(*) (const char *)``.

        This is a GNU extension.

    ``gl_stat``

        The address of an alternative implementation of the ``stat64`` function to
        get information about an object in the filesystem. It is used if the ``GLOB_
        ALTDIRFUNC`` bit is set in the flag parameter. The type of this field is
        ``int (*) (const char *, struct stat64 *)``.

        This is a GNU extension.

    ``gl_lstat``

        The address of an alternative implementation of the ``lstat64`` function to
        get information about an object in the filesystems, not following symbolic
        links. It is used if the ``GLOB_ALTDIRFUNC`` bit is set in the flag parameter.
        The type of this field is ``int (*) (const char *, struct stat64 *)``.

        This is a GNU extension.

    ``gl_flags``

        The flags used when ``glob`` was called. In addition, ``GLOB_MAGCHAR`` might
        be set. See Section [Flags for Globbing], for more details.

        This is a GNU extension.

``int glob ( const char *pattern, int flags, int ( *errfunc ) ( const
char *filename, int error-code ) , glob t *vector-ptr )``        [Function]

    Preliminary: | MT-Unsafe race:utent env sig:ALRM timer locale | AS-Unsafe dlopen
    plugin corrupt heap lock | AC-Unsafe corrupt lock fd mem | 
        See Section [POSIX Safety Concepts].



    The function ``glob`` does globbing using the pattern ``pattern`` in the current directory.
    It puts the result in a newly allocated vector, and stores the size and address of
    this vector into ``*vector-ptr``. The argument ``flags`` is a combination of bit flags; see
    Section [Flags for Globbing] for details of the flags.

    The result of globbing is a sequence of file names. The function ``glob`` allocates a string
    for each resulting word, then allocates a vector of type ``char **`` to store the addresses
    of these strings. The last element of the vector is a null pointer. This vector is called
    the ``word vector``.

    To return this vector, ``glob`` stores both its address and its length (number of elements,
    not counting the terminating null pointer) into ``*vector-ptr``.

    Normally, ``glob`` sorts the file names alphabetically before returning them. You can
    turn this off with the flag ``GLOB_NOSORT`` if you want to get the information as fast
    as possible. Usually it’s a good idea to let ``glob`` sort them—if you process the files
    in alphabetical order, the users will have a feel for the rate of progress that your
    application is making.

    If ``glob`` succeeds, it returns ``0``. Otherwise, it returns one of these error codes:

    ``GLOB_ABORTED``

        There was an error opening a directory, and you used the flag ``GLOB_ERR``
        or your specified ``errfunc`` returned a nonzero value. See below for an
        explanation of the ``GLOB_ERR`` flag and ``errfunc``.

    ``GLOB_NOMATCH``

        The pattern didn’t match any existing files. If you use the ``GLOB_NOCHECK``
        flag, then you never get this error code, because that flag tells ``glob`` to
        ``pretend`` that the pattern matched at least one file.

    ``GLOB_NOSPACE``

        It was impossible to allocate memory to hold the result.

    In the event of an error, ``glob`` stores information in ``*vector-ptr`` about all the matches
    it has found so far.

    It is important to notice that the ``glob`` function will not fail if it encounters directories
    or files which cannot be handled without the ``LFS`` interfaces. The implementation of
    ``glob`` is supposed to use these functions internally. This at least is the assumption
    made by the Unix standard. The GNU extension of allowing the user to provide their
    own directory handling and ``stat`` functions complicates things a bit. If these callback
    functions are used and a large file or directory is encountered ``glob can fail``.


``int glob64 ( const char *pattern, int flags, int ( *errfunc ) ( const
char *filename, int error-code ) , glob64 t *vector-ptr )            [Function]

    Preliminary: | MT-Unsafe race:utent env sig:ALRM timer locale | AS-Unsafe dlopen
    corrupt heap lock | AC-Unsafe corrupt lock fd mem |
        See Section [POSIX Safety Concepts]

    The ``glob64`` function was added as part of the ``Large File Summit`` extensions but is not
    part of the original ``LFS`` proposal. The reason for this is simple: it is not necessary.
    The necessity for a ``glob64`` function is added by the extensions of the GNU ``glob``
    implementation which allows the user to provide their own directory handling and
    stat functions. The ``readdir`` and ``stat`` functions do depend on the choice of ``_FILE_
    OFFSET_BITS`` since the definition of the types ``struct dirent`` and ``struct stat`` will
    change depending on the choice.

    Besides this difference, ``glob64`` works just like ``glob`` in all aspects.

    This function is a GNU extension.

Flags for Globbing
------------------

This section describes the standard flags that you can specify in the ``flags`` argument to ``glob``.
Choose the flags you want, and combine them with the C bitwise OR operator |.

Note that there are Section [More Flags for Globbing] available as GNU extensions.
