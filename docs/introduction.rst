=============
Introduction
=============

The C language provides no built-in facilities for performing such common operations as
input/output, memory management, string manipulation, and the like. Instead, these facilities
are defined in a standard library, which you compile and link with your programs.

The GNU C Library, described in this document, defines all of the library functions that
are specified by the ISO C standard, as well as additional features specific to POSIX and
other derivatives of the Unix operating system, and extensions specific to GNU systems.
The purpose of this manual is to tell you how to use the facilities of the GNU C Library.
We have mentioned which features belong to which standards to help you identify things
that are potentially non-portable to other systems. But the emphasis in this manual is not
on strict portability.

Getting Started
===============

This manual is written with the assumption that you are at least somewhat familiar with
the C programming language and basic programming concepts. Specifically, familiarity
with ISO standard C (see Section 1.2.1 [ISO C], page 2), rather than “traditional” pre-ISO
C dialects, is assumed.

The GNU C Library includes several header files, each of which provides definitions and
declarations for a group of related facilities; this information is used by the C compiler
when processing your program. For example, the header file stdio.h declares facilities
for performing input and output, and the header file string.h declares string processing
utilities. The organization of this manual generally follows the same division as the header
files.

If you are reading this manual for the first time, you should read all of the introductory
material and skim the remaining chapters. There are a lot of functions in the GNU C
Library and it’s not realistic to expect that you will be able to remember exactly how to
use each and every one of them. It’s more important to become generally familiar with the
kinds of facilities that the library provides, so that when you are writing your programs you
can recognize when to make use of library functions, and where in this manual you can find
more specific information about them.

Standards and Portability
==========================

This section discusses the various standards and other sources that the GNU C Library is
based upon. These sources include the ISO C and POSIX standards, and the System V
and Berkeley Unix implementations.

The primary focus of this manual is to tell you how to make effective use of the GNU C
Library facilities. But if you are concerned about making your programs compatible with
these standards, or portable to operating systems other than GNU, this can affect how you
use the library. This section gives you an overview of these standards, so that you will know
what they are when they are mentioned in other parts of the manual.

See Appendix B [Summary of Library Facilities], page 897, for an alphabetical list of the
functions and other symbols provided by the library. This list also states which standards
each function or symbol comes from.

ISO C
-----

The GNU C Library is compatible with the C standard adopted by the American National
Standards Institute (ANSI): American National Standard X3.159-1989—“ANSI C”
and later by the International Standardization Organization (ISO): ISO/IEC 9899:1990,
“Programming languages—C”. We here refer to the standard as ISO C since this is the
more general standard in respect of ratification. The header files and library facilities that
make up the GNU C Library are a superset of those specified by the ISO C standard.

If you are concerned about strict adherence to the ISO C standard, you should use the
‘-ansi’ option when you compile your programs with the GNU C compiler. This tells
the compiler to define only ISO standard features from the library header files, unless you
explicitly ask for additional features. See Section 1.3.4 [Feature Test Macros], page 15, for
information on how to do this

Being able to restrict the library to include only ISO C features is important because
ISO C puts limitations on what names can be defined by the library implementation, and
the GNU extensions don’t fit these limitations. See Section 1.3.3 [Reserved Names], page 14,
for more information about these restrictions.

This manual does not attempt to give you complete details on the differences between
ISO C and older dialects. It gives advice on how to write programs to work portably under
multiple C dialects, but does not aim for completeness.

POSIX (The Portable Operating System Interface)
-----------------------------------------------

The library facilities specified by the POSIX standards are a superset of those required
by ISO C; POSIX specifies additional features for ISO C functions, as well as specifying
new additional functions. In general, the additional requirements and functionality defined
by the POSIX standards are aimed at providing lower-level support for a particular kind of
operating system environment, rather than general programming language support which
can run in many diverse operating system environments.

The GNU C Library implements all of the functions specified in ISO/IEC 9945-1:1996,
the POSIX System Application Program Interface, commonly referred to as POSIX.1. The
primary extensions to the ISO C facilities specified by this standard include file system
interface primitives (see Chapter 14 [File System Interface], page 379), device-specific terminal
control functions (see Chapter 17 [Low-Level Terminal Interface], page 479), and
process control functions (see Chapter 26 [Processes], page 752).

Some facilities from ISO/IEC 9945-2:1993, the POSIX Shell and Utilities standard
(POSIX.2) are also implemented in the GNU C Library. These include utilities for dealing
with regular expressions and other pattern matching facilities (see Chapter 10 [Pattern
Matching], page 223).

POSIX Safety Concepts
######################

This manual documents various safety properties of GNU C Library functions, in lines that
follow their prototypes and look like:

Preliminary: | MT-Safe | AS-Safe | AC-Safe

The properties are assessed according to the criteria set forth in the POSIX standard for
such safety contexts as Thread-, Async-Signal- and Async-Cancel- -Safety. Intuitive definitions
of these properties, attempting to capture the meaning of the standard definitions,
follow.

- MT-Safe or Thread-Safe functions are safe to call in the presence of other threads. MT,
  in MT-Safe, stands for Multi Thread.
  Being MT-Safe does not imply a function is atomic, nor that it uses any of the memory
  synchronization mechanisms POSIX exposes to users. It is even possible that calling
  MT-Safe functions in sequence does not yield an MT-Safe combination. For example,
  having a thread call two MT-Safe functions one right after the other does not guarantee
  behavior equivalent to atomic execution of a combination of both functions, since
  concurrent calls in other threads may interfere in a destructive way.
  Whole-program optimizations that could inline functions across library interfaces may
  expose unsafe reordering, and so performing inlining across the GNU C Library interface
  is not recommended. The documented MT-Safety status is not guaranteed under
  whole-program optimization. However, functions defined in user-visible headers are
  designed to be safe for inlining.

- AS-Safe or Async-Signal-Safe functions are safe to call from asynchronous signal handlers.
  AS, in AS-Safe, stands for Asynchronous Signal.
  Many functions that are AS-Safe may set errno, or modify the floating-point environment,
  because their doing so does not make them unsuitable for use in signal handlers.
  However, programs could misbehave should asynchronous signal handlers modify this
  thread-local state, and the signal handling machinery cannot be counted on to preserve
  it. Therefore, signal handlers that call functions that may set errno or modify
  the floating-point environment must save their original values, and restore them before
  returning.

- AC-Safe or Async-Cancel-Safe functions are safe to call when asynchronous cancellation
  is enabled. AC in AC-Safe stands for Asynchronous Cancellation.
  The POSIX standard defines only three functions to be AC-Safe, namely pthread_
  cancel, pthread_setcancelstate, and pthread_setcanceltype. At present the
  GNU C Library provides no guarantees beyond these three functions, but does document
  which functions are presently AC-Safe. This documentation is provided for use
  by the GNU C Library developers.
  Just like signal handlers, cancellation cleanup routines must configure the floating point
  environment they require. The routines cannot assume a floating point environment,
  particularly when asynchronous cancellation is enabled. If the configuration of the
  floating point environment cannot be performed atomically then it is also possible that
  the environment encountered is internally inconsistent.

- MT-Unsafe, AS-Unsafe, AC-Unsafe functions are not safe to call within the safety contexts
  described above. Calling them within such contexts invokes undefined behavior.
  Functions not explicitly documented as safe in a safety context should be regarded as
  Unsafe.

- Preliminary safety properties are documented, indicating these properties may not be
  counted on in future releases of the GNU C Library.

Such preliminary properties are the result of an assessment of the properties of our
current implementation, rather than of what is mandated and permitted by current
and future standards.

Although we strive to abide by the standards, in some cases our implementation is safe
even when the standard does not demand safety, and in other cases our implementation
does not meet the standard safety requirements. The latter are most likely bugs; the
former, when marked as Preliminary, should not be counted on: future standards may
require changes that are not compatible with the additional safety properties afforded
by the current implementation

Furthermore, the POSIX standard does not offer a detailed definition of safety. We
assume that, by “safe to call”, POSIX means that, as long as the program does not
invoke undefined behavior, the “safe to call” function behaves as specified, and does
not cause other functions to deviate from their specified behavior. We have chosen to
use its loose definitions of safety, not because they are the best definitions to use, but
because choosing them harmonizes this manual with POSIX


