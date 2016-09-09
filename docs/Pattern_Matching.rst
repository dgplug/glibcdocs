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


