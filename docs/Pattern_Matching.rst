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
                  See Section 1.2.2.1 [POSIX Safety Concepts], page 2.

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
    mend this name because we don’t use the term "pathname" for file names.


