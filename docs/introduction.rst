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

Over time, we envision evolving the preliminary safety notes into stable commitments,
as stable as those of our interfaces. As we do, we will remove the Preliminary keyword
from safety notes. As long as the keyword remains, however, they are not to be regarded
as a promise of future behavior.

Other keywords that appear in safety notes are defined in subsequent sections.

Unsafe Features
################

Functions that are unsafe to call in certain contexts are annotated with keywords that
document their features that make them unsafe to call. AS-Unsafe features in this section
indicate the functions are never safe to call when asynchronous signals are enabled.
AC-Unsafe features indicate they are never safe to call when asynchronous cancellation is
enabled. There are no MT-Unsafe marks in this section.

- lock
  Functions marked with lock as an AS-Unsafe feature may be interrupted by a signal
  while holding a non-recursive lock. If the signal handler calls another such function
  that takes the same lock, the result is a deadlock.
  Functions annotated with lock as an AC-Unsafe feature may, if cancelled
  asynchronously, fail to release a lock that would have been released if their execution
  had not been interrupted by asynchronous thread cancellation. Once a lock is left
  taken, attempts to take that lock will block indefinitely.

- corrupt
  Functions marked with corrupt as an AS-Unsafe feature may corrupt data structures
  and misbehave when they interrupt, or are interrupted by, another such function.
  Unlike functions marked with lock, these take recursive locks to avoid MT-Safety
  problems, but this is not enough to stop a signal handler from observing a partiallyupdated
  data structure. Further corruption may arise from the interrupted function’s
  failure to notice updates made by signal handlers.
  Functions marked with corrupt as an AC-Unsafe feature may leave data structures in a
  corrupt, partially updated state. Subsequent uses of the data structure may misbehave.

- heap
  Functions marked with heap may call heap memory management functions from the
  malloc/free family of functions and are only as safe as those functions. This note is
  thus equivalent to:
  | AS-Unsafe lock | AC-Unsafe lock fd mem |

- dlopen
  Functions marked with dlopen use the dynamic loader to load shared libraries into
  the current execution image. This involves opening files, mapping them into memory,
  allocating additional memory, resolving symbols, applying relocations and more, all of
  this while holding internal dynamic loader locks.
  The locks are enough for these functions to be AS- and AC-Unsafe, but other issues
  may arise. At present this is a placeholder for all potential safety issues raised by
  dlopen.

- plugin
  Functions annotated with plugin may run code from plugins that may be external to
  the GNU C Library. Such plugin functions are assumed to be MT-Safe, AS-Unsafe
  and AC-Unsafe. Examples of such plugins are stack unwinding libraries, name service
  switch (NSS) and character set conversion (iconv) back-ends.
  Although the plugins mentioned as examples are all brought in by means of dlopen,
  the plugin keyword does not imply any direct involvement of the dynamic loader or
  the libdl interfaces, those are covered by dlopen. For example, if one function loads a
  module and finds the addresses of some of its functions, while another just calls those
  already-resolved functions, the former will be marked with dlopen, whereas the latter
  will get the plugin. When a single function takes all of these actions, then it gets both
  marks.

- i18n
  Functions marked with i18n may call internationalization functions of the gettext
  family and will be only as safe as those functions. This note is thus equivalent to:
  | MT-Safe env | AS-Unsafe corrupt heap dlopen | AC-Unsafe corrupt |

- timer
  Functions marked with timer use the alarm function or similar to set a time-out for a
  system call or a long-running operation. In a multi-threaded program, there is a risk
  that the time-out signal will be delivered to a different thread, thus failing to interrupt
  the intended thread. Besides being MT-Unsafe, such functions are always AS-Unsafe,
  because calling them in signal handlers may interfere with timers set in the interrupted
  code, and AC-Unsafe, because there is no safe way to guarantee an earlier timer will
  be reset in case of asynchronous cancellation.

Conditionally Safe Features
############################

For some features that make functions unsafe to call in certain contexts, there are known
ways to avoid the safety problem other than refraining from calling the function altogether.
The keywords that follow refer to such features, and each of their definitions indicate how
the whole program needs to be constrained in order to remove the safety problem indicated
by the keyword. Only when all the reasons that make a function unsafe are observed and
addressed, by applying the documented constraints, does the function become safe to call
in a context.

- init
  Functions marked with init as an MT-Unsafe feature perform MT-Unsafe initialization
  when they are first called.
  Calling such a function at least once in single-threaded mode removes this specific cause
  for the function to be regarded as MT-Unsafe. If no other cause for that remains, the
  function can then be safely called after other threads are started.
  Functions marked with init as an AS- or AC-Unsafe feature use the internal libc_
  once machinery or similar to initialize internal data structures.
  If a signal handler interrupts such an initializer, and calls any function that also performs
  libc_once initialization, it will deadlock if the thread library has been loaded.
  Furthermore, if an initializer is partially complete before it is canceled or interrupted
  by a signal whose handler requires the same initialization, some or all of the initialization
  may be performed more than once, leaking resources or even resulting in corrupt
  internal data.
  Applications that need to call functions marked with init as an AS- or AC-Unsafe
  feature should ensure the initialization is performed before configuring signal handlers
  or enabling cancellation, so that the AS- and AC-Safety issues related with libc_once
  do not arise.

- race
  Functions annotated with race as an MT-Safety issue operate on objects in ways that
  may cause data races or similar forms of destructive interference out of concurrent
  execution. In some cases, the objects are passed to the functions by users; in others,
  they are used by the functions to return values to users; in others, they are not even
  exposed to users.
  We consider access to objects passed as (indirect) arguments to functions to be data
  race free. The assurance of data race free objects is the caller’s responsibility. We
  will not mark a function as MT-Unsafe or AS-Unsafe if it misbehaves when users fail
  to take the measures required by POSIX to avoid data races when dealing with such
  objects. As a general rule, if a function is documented as reading from an object
  passed (by reference) to it, or modifying it, users ought to use memory synchronization
  primitives to avoid data races just as they would should they perform the accesses
  themselves rather than by calling the library function. FILE streams are the exception
  to the general rule, in that POSIX mandates the library to guard against data races
  in many functions that manipulate objects of this specific opaque type. We regard
  this as a convenience provided to users, rather than as a general requirement whose
  expectations should extend to other types.
  In order to remind users that guarding certain arguments is their responsibility, we will
  annotate functions that take objects of certain types as arguments. We draw the line
  for objects passed by users as follows: objects whose types are exposed to users, and
  that users are expected to access directly, such as memory buffers, strings, and various
  user-visible struct types, do not give reason for functions to be annotated with race.
  It would be noisy and redundant with the general requirement, and not many would
  be surprised by the library’s lack of internal guards when accessing objects that can be
  accessed directly by users.
  As for objects that are opaque or opaque-like, in that they are to be manipulated only
  by passing them to library functions (e.g., FILE, DIR, obstack, iconv_t), there might
  be additional expectations as to internal coordination of access by the library. We will
  annotate, with race followed by a colon and the argument name, functions that take
  such objects but that do not take care of synchronizing access to them by default. For
  example, FILE stream unlocked functions will be annotated, but those that perform
  implicit locking on FILE streams by default will not, even though the implicit locking
  may be disabled on a per-stream basis.
  In either case, we will not regard as MT-Unsafe functions that may access user-supplied
  objects in unsafe ways should users fail to ensure the accesses are well defined. The
  notion prevails that users are expected to safeguard against data races any user-supplied
  objects that the library accesses on their behalf.
  This user responsibility does not apply, however, to objects controlled by the library
  itself, such as internal objects and static buffers used to return values from certain
  calls. When the library doesn’t guard them against concurrent uses, these cases are
  regarded as MT-Unsafe and AS-Unsafe (although the race mark under AS-Unsafe will
  be omitted as redundant with the one under MT-Unsafe). As in the case of userexposed
  objects, the mark may be followed by a colon and an identifier. The identifier
  groups all functions that operate on a certain unguarded object; users may avoid the
  MT-Safety issues related with unguarded concurrent access to such internal objects
  by creating a non-recursive mutex related with the identifier, and always holding the
  mutex when calling any function marked as racy on that identifier, as they would have
  to should the identifier be an object under user control. The non-recursive mutex
  avoids the MT-Safety issue, but it trades one AS-Safety issue for another, so use in
  asynchronous signals remains undefined.
  When the identifier relates to a static buffer used to hold return values, the mutex
  must be held for as long as the buffer remains in use by the caller. Many functions
  that return pointers to static buffers offer reentrant variants that store return values in
  caller-supplied buffers instead. In some cases, such as tmpname, the variant is chosen
  not by calling an alternate entry point, but by passing a non-NULL pointer to the buffer
  in which the returned values are to be stored. These variants are generally preferable
  in multi-threaded programs, although some of them are not MT-Safe because of other
  internal buffers, also documented with race notes.

- const
  Functions marked with const as an MT-Safety issue non-atomically modify internal
  objects that are better regarded as constant, because a substantial portion of the
  GNU C Library accesses them without synchronization. Unlike race, that causes both
  readers and writers of internal objects to be regarded as MT-Unsafe and AS-Unsafe, this
  mark is applied to writers only. Writers remain equally MT- and AS-Unsafe to call, but
  the then-mandatory constness of objects they modify enables readers to be regarded as
  MT-Safe and AS-Safe (as long as no other reasons for them to be unsafe remain), since
  the lack of synchronization is not a problem when the objects are effectively constant.
  The identifier that follows the const mark will appear by itself as a safety note in
  readers. Programs that wish to work around this safety issue, so as to call writers,
  may use a non-recursve rwlock associated with the identifier, and guard all calls to
  functions marked with const followed by the identifier with a write lock, and all calls to
  functions marked with the identifier by itself with a read lock. The non-recursive locking
  removes the MT-Safety problem, but it trades one AS-Safety problem for another, so
  use in asynchronous signals remains undefined.

- sig
  Functions marked with sig as a MT-Safety issue (that implies an identical AS-Safety issue,
  omitted for brevity) may temporarily install a signal handler for internal purposes,
  which may interfere with other uses of the signal, identified after a colon.
  This safety problem can be worked around by ensuring that no other uses of the signal
  will take place for the duration of the call. Holding a non-recursive mutex while calling
  all functions that use the same temporary signal; blocking that signal before the call
  and resetting its handler afterwards is recommended.
  There is no safe way to guarantee the original signal handler is restored in case of
  asynchronous cancellation, therefore so-marked functions are also AC-Unsafe.
  Besides the measures recommended to work around the MT- and AS-Safety problem,
  in order to avert the cancellation problem, disabling asynchronous cancellation and
  installing a cleanup handler to restore the signal to the desired state and to release the
  mutex are recommended.

- term
  Functions marked with term as an MT-Safety issue may change the terminal settings
  in the recommended way, namely: call tcgetattr, modify some flags, and then call
  tcsetattr; this creates a window in which changes made by other threads are lost.
  Thus, functions marked with term are MT-Unsafe. The same window enables changes
  made by asynchronous signals to be lost. These functions are also AS-Unsafe, but the
  corresponding mark is omitted as redundant.
  It is thus advisable for applications using the terminal to avoid concurrent and reentrant
  interactions with it, by not using it in signal handlers or blocking signals that
  might use it, and holding a lock while calling these functions and interacting with the
  terminal. This lock should also be used for mutual exclusion with functions marked
  with race:tcattr(fd), where fd is a file descriptor for the controlling terminal. The
  caller may use a single mutex for simplicity, or use one mutex per terminal, even if
  referenced by different file descriptors.
  Functions marked with term as an AC-Safety issue are supposed to restore terminal
  settings to their original state, after temporarily changing them, but they may fail to
  do so if cancelled.
  Besides the measures recommended to work around the MT- and AS-Safety problem,
  in order to avert the cancellation problem, disabling asynchronous cancellation and
  installing a cleanup handler to restore the terminal settings to the original state and
  to release the mutex are recommended.

Other Safety Remarks
#######################

Additional keywords may be attached to functions, indicating features that do not make
a function unsafe to call, but that may need to be taken into account in certain classes of
programs:

- locale
  Functions annotated with locale as an MT-Safety issue read from the locale object
  without any form of synchronization. Functions annotated with locale called concurrently
  with locale changes may behave in ways that do not correspond to any of the
  locales active during their execution, but an unpredictable mix thereof.
  We do not mark these functions as MT- or AS-Unsafe, however, because functions
  that modify the locale object are marked with const:locale and regarded as unsafe.
  Being unsafe, the latter are not to be called when multiple threads are running or asynchronous
  signals are enabled, and so the locale can be considered effectively constant
  in these contexts, which makes the former safe.

- env
  Functions marked with env as an MT-Safety issue access the environment with getenv
  or similar, without any guards to ensure safety in the presence of concurrent modifications.
  We do not mark these functions as MT- or AS-Unsafe, however, because functions
  that modify the environment are all marked with const:env and regarded as unsafe.
  Being unsafe, the latter are not to be called when multiple threads are running or
  asynchronous signals are enabled, and so the environment can be considered effectively
  constant in these contexts, which makes the former safe.

- hostid
  The function marked with hostid as an MT-Safety issue reads from the system-wide
  data structures that hold the “host ID” of the machine. These data structures cannot
  generally be modified atomically. Since it is expected that the “host ID” will not normally
  change, the function that reads from it (gethostid) is regarded as safe, whereas
  the function that modifies it (sethostid) is marked with const:hostid, indicating
  it may require special care if it is to be called. In this specific case, the special care
  amounts to system-wide (not merely intra-process) coordination.

- sigintr
  Functions marked with sigintr as an MT-Safety issue access the _sigintr internal
  data structure without any guards to ensure safety in the presence of concurrent modifications.
  We do not mark these functions as MT- or AS-Unsafe, however, because functions that
  modify the this data structure are all marked with const:sigintr and regarded as
  unsafe. Being unsafe, the latter are not to be called when multiple threads are running
  or asynchronous signals are enabled, and so the data structure can be considered
  effectively constant in these contexts, which makes the former saf

- fd
  Functions annotated with fd as an AC-Safety issue may leak file descriptors if asynchronous
  thread cancellation interrupts their execution. Functions that allocate or deallocate file descriptors will generally be marked as such.
  Even if they attempted to protect the file descriptor allocation and deallocation with
  cleanup regions, allocating a new descriptor and storing its number where the cleanup
  region could release it cannot be performed as a single atomic operation. Similarly,
  releasing the descriptor and taking it out of the data structure normally responsible for
  releasing it cannot be performed atomically. There will always be a window in which
  the descriptor cannot be released because it was not stored in the cleanup handler
  argument yet, or it was already taken out before releasing it. It cannot be taken out
  after release: an open descriptor could mean either that the descriptor still has to be
  closed, or that it already did so but the descriptor was reallocated by another thread
  or signal handler.
  Such leaks could be internally avoided, with some performance penalty, by temporarily
  disabling asynchronous thread cancellation. However, since callers of allocation or
  deallocation functions would have to do this themselves, to avoid the same sort of leak
  in their own layer, it makes more sense for the library to assume they are taking care of
  it than to impose a performance penalty that is redundant when the problem is solved
  in upper layers, and insufficient when it is not.
  This remark by itself does not cause a function to be regarded as AC-Unsafe. However,
  cumulative effects of such leaks may pose a problem for some programs. If this is the
  case, suspending asynchronous cancellation for the duration of calls to such functions
  is recommended.

- mem
  Functions annotated with mem as an AC-Safety issue may leak memory if asynchronous
  thread cancellation interrupts their execution.
  The problem is similar to that of file descriptors: there is no atomic interface to allocate
  memory and store its address in the argument to a cleanup handler, or to release it
  and remove its address from that argument, without at least temporarily disabling
  asynchronous cancellation, which these functions do not do.
  This remark does not by itself cause a function to be regarded as generally AC-Unsafe.
  However, cumulative effects of such leaks may be severe enough for some programs that
  disabling asynchronous cancellation for the duration of calls to such functions may be
  required.

- cwd
  Functions marked with cwd as an MT-Safety issue may temporarily change the current
  working directory during their execution, which may cause relative pathnames
  to be resolved in unexpected ways in other threads or within asynchronous signal or
  cancellation handlers.
  This is not enough of a reason to mark so-marked functions as MT- or AS-Unsafe, but
  when this behavior is optional (e.g., nftw with FTW_CHDIR), avoiding the option may
  be a good alternative to using full pathnames or file descriptor-relative (e.g. openat)
  system calls.

- !posix
  This remark, as an MT-, AS- or AC-Safety note to a function, indicates the safety status
  of the function is known to differ from the specified status in the POSIX standard. For example, POSIX does not require a function to be Safe, but our implementation is, or
  vice-versa.
  For the time being, the absence of this remark does not imply the safety properties we
  documented are identical to those mandated by POSIX for the corresponding functions.

- :identifier
  Annotations may sometimes be followed by identifiers, intended to group several functions
  that e.g. access the data structures in an unsafe way, as in race and const, or to
  provide more specific information, such as naming a signal in a function marked with
  sig. It is envisioned that it may be applied to lock and corrupt as well in the future.
  In most cases, the identifier will name a set of functions, but it may name global objects
  or function arguments, or identifiable properties or logical components associated with
  them, with a notation such as e.g. :buf(arg) to denote a buffer associated with the
  argument arg, or :tcattr(fd) to denote the terminal attributes of a file descriptor fd.
  The most common use for identifiers is to provide logical groups of functions and
  arguments that need to be protected by the same synchronization primitive in order
  to ensure safe operation in a given context.

- /condition
  Some safety annotations may be conditional, in that they only apply if a boolean
  expression involving arguments, global variables or even the underlying kernel evaluates
  to true. Such conditions as /hurd or /!linux!bsd indicate the preceding marker only
  applies when the underlying kernel is the HURD, or when it is neither Linux nor a
  BSD kernel, respectively. /!ps and /one_per_line indicate the preceding marker
  only applies when argument ps is NULL, or global variable one per line is nonzero.
  When all marks that render a function unsafe are adorned with such conditions, and
  none of the named conditions hold, then the function can be regarded as safe.

Berkeley Unix
--------------

The GNU C Library defines facilities from some versions of Unix which are not formally
standardized, specifically from the 4.2 BSD, 4.3 BSD, and 4.4 BSD Unix systems (also
known as Berkeley Unix) and from SunOS (a popular 4.2 BSD derivative that includes
some Unix System V functionality). These systems support most of the ISO C and POSIX
facilities, and 4.4 BSD and newer releases of SunOS in fact support them all.

The BSD facilities include symbolic links (see Section 14.5 [Symbolic Links], page 395),
the select function (see Section 13.8 [Waiting for Input or Output], page 344), the BSD
signal functions (see Section 24.10 [BSD Signal Handling], page 706), and sockets (see
Chapter 16 [Sockets], page 431).

SVID (The System V Interface Description)
------------------------------------------

The System V Interface Description (SVID) is a document describing the AT&T Unix
System V operating system. It is to some extent a superset of the POSIX standard (see
Section 1.2.2 [POSIX (The Portable Operating System Interface)], page 2).

The GNU C Library defines most of the facilities required by the SVID that are not
also required by the ISO C or POSIX standards, for compatibility with System V Unix and
other Unix systems (such as SunOS) which include these facilities. However, many of the
more obscure and less generally useful facilities required by the SVID are not included. (In
fact, Unix System V itself does not provide them all.)

The supported facilities from System V include the methods for inter-process communication
and shared memory, the hsearch and drand48 families of functions, fmtmsg and
several of the mathematical functions.

XPG (The X/Open Portability Guide)
-----------------------------------

The X/Open Portability Guide, published by the X/Open Company, Ltd., is a more general
standard than POSIX. X/Open owns the Unix copyright and the XPG specifies the
requirements for systems which are intended to be a Unix system.

The GNU C Library complies to the X/Open Portability Guide, Issue 4.2, with all extensions
common to XSI (X/Open System Interface) compliant systems and also all X/Open
UNIX extensions.

The additions on top of POSIX are mainly derived from functionality available in
System V and BSD systems. Some of the really bad mistakes in System V systems were
corrected, though. Since fulfilling the XPG standard with the Unix extensions is a precondition
for getting the Unix brand chances are good that the functionality is available on
commercial systems.

Using the Library
=================

This section describes some of the practical issues involved in using the GNU C Library.

Header Files
-------------

Libraries for use by C programs really consist of two parts: header files that define types and
macros and declare variables and functions; and the actual library or archive that contains
the definitions of the variables and functions.

(Recall that in C, a declaration merely provides information that a function or variable
exists and gives its type. For a function declaration, information about the types of its
arguments might be provided as well. The purpose of declarations is to allow the compiler
to correctly process references to the declared variables and functions. A definition, on the
other hand, actually allocates storage for a variable or says what a function does.)

In order to use the facilities in the GNU C Library, you should be sure that your program
source files include the appropriate header files. This is so that the compiler has declarations
of these facilities available and can correctly process references to them. Once your program
has been compiled, the linker resolves these references to the actual definitions provided in
the archive file.

Header files are included into a program source file by the ‘#include’ preprocessor
directive. The C language supports two forms of this directive; the firs
::

    #include "header"

is typically used to include a header file header that you write yourself; this would contain
definitions and declarations describing the interfaces between the different parts of your
particular application. By contrast,
::

    #include <file.h>

is typically used to include a header file file.h that contains definitions and declarations
for a standard library. This file would normally be installed in a standard place by your
system administrator. You should use this second form for the C library header files.

Typically, ‘#include’ directives are placed at the top of the C source file, before any
other code. If you begin your source files with some comments explaining what the code in
the file does (a good idea), put the ‘#include’ directives immediately afterwards, following
the feature test macro definition (see Section 1.3.4 [Feature Test Macros], page 15).

For more information about the use of header files and ‘#include’ directives, see Section
“Header Files” in *The GNU C Preprocessor Manual*.

The GNU C Library provides several header files, each of which contains the type and
macro definitions and variable and function declarations for a group of related facilities.
This means that your programs may need to include several header files, depending on
exactly which facilities you are using.

Some library header files include other library header files automatically. However, as a
matter of programming style, you should not rely on this; it is better to explicitly include all
the header files required for the library facilities you are using. The GNU C Library header
files have been written in such a way that it doesn’t matter if a header file is accidentally
included more than once; including a header file a second time has no effect. Likewise, if
your program needs to include multiple header files, the order in which they are included
doesn’t matter.

**Compatibility Note:** Inclusion of standard header files in any order and any number of
times works in any ISO C implementation. However, this has traditionally not been the
case in many older C implementations.

Strictly speaking, you don’t have to include a header file to use a function it declares;
you could declare the function explicitly yourself, according to the specifications in this
manual. But it is usually better to include the header file because it may define types and
macros that are not otherwise available and because it may define more efficient macro
replacements for some functions. It is also a sure way to have the correct declaration.

Macro Definitions of Functions
--------------------------------

If we describe something as a function in this manual, it may have a macro definition as
well. This normally has no effect on how your program runs—the macro definition does
the same thing as the function would. In particular, macro equivalents for library functions
evaluate arguments exactly once, in the same way that a function call would. The main
reason for these macro definitions is that sometimes they can produce an inline expansion
that is considerably faster than an actual function call.

Taking the address of a library function works even if it is also defined as a macro. This
is because, in this context, the name of the function isn’t followed by the left parenthesis
that is syntactically necessary to recognize a macro call.

You might occasionally want to avoid using the macro definition of a function—perhaps
to make your program easier to debug. There are two ways you can do this:

- You can avoid a macro definition in a specific use by enclosing the name of the function
  in parentheses. This works because the name of the function don’t appear in a
  syntactic context where it is recognizable as a macro call.

- You can suppress any macro definition for a whole source file by using the ‘#undef’
  preprocessor directive, unless otherwise stated explicitly in the description of that facility.
  For example, suppose the header file stdlib.h declares a function named abs with
  ::

        extern int abs (int);

and also provides a macro definition for abs. Then, in:
::

    #include <stdlib.h>
    int f (int *i) { return abs (++*i); }

the reference to abs might refer to either a macro or a function. On the other hand, in each
of the following examples the reference is to a function and not a macro.
::

    #include <stdlib.h>
    int g (int *i) { return (abs) (++*i); }
    #undef abs
    int h (int *i) { return abs (++*i); }

Since macro definitions that double for a function behave in exactly the same way as the
actual function version, there is usually no need for any of these methods. In fact, removing
macro definitions usually just makes your program slower.

Reserved Names
--------------

The names of all library types, macros, variables and functions that come from the ISO C
standard are reserved unconditionally; your program may not redefine these names. All
other library names are reserved if your program explicitly includes the header file that
defines or declares them. There are several reasons for these restrictions:

- Other people reading your code could get very confused if you were using a function
  named exit to do something completely different from what the standard exit function
  does, for example. Preventing this situation helps to make your programs easier to
  understand and contributes to modularity and maintainability.

- It avoids the possibility of a user accidentally redefining a library function that is called
  by other library functions. If redefinition were allowed, those other functions would not
  work properly.

- It allows the compiler to do whatever special optimizations it pleases on calls to these
  functions, without the possibility that they may have been redefined by the user. Some
  library facilities, such as those for dealing with variadic arguments (see Section A.2
  [Variadic Functions], page 882) and non-local exits (see Chapter 23 [Non-Local Exits],
  page 655), actually require a considerable amount of cooperation on the part of the C
  compiler, and with respect to the implementation, it might be easier for the compiler
  to treat these as built-in parts of the language.

In addition to the names documented in this manual, reserved names include all external
identifiers (global functions and variables) that begin with an underscore (‘_’) and all identifiers
regardless of use that begin with either two underscores or an underscore followed by
a capital letter are reserved names. This is so that the library and header files can define
functions, variables, and macros for internal purposes without risk of conflict with names
in user programs.

Some additional classes of identifier names are reserved for future extensions
to the C language or the POSIX.1 environment. While using these names for your
own purposes right now might not cause a problem, they do raise the possibility
of conflict with future versions of the C or POSIX standards, so you should
avoid these names.

- Names beginning with a capital ‘E’ followed a digit or uppercase letter may be used for
  additional error code names. See Chapter 2 [Error Reporting], page 22.

- Names that begin with either ‘is’ or ‘to’ followed by a lowercase letter may be used
  for additional character testing and conversion functions. See Chapter 4 [Character
  Handling], page 76.

- Names that begin with ‘LC_’ followed by an uppercase letter may be used for additional
  macros specifying locale attributes. See Chapter 7 [Locales and Internationalization],
  page 169.

- Names of all existing mathematics functions (see Chapter 19 [Mathematics], page 514)
  suffixed with ‘f’ or ‘l’ are reserved for corresponding functions that operate on float

- Names that begin with ‘SIG’ followed by an uppercase letter are reserved for additional
  signal names. See Section 24.2 [Standard Signals], page 666.and long double arguments, respectively.

- Names that begin with ‘SIG_’ followed by an uppercase letter are reserved for additional
 signal actions. See Section 24.3.1 [Basic Signal Handling], page 675.

- Names beginning with ‘str’, ‘mem’, or ‘wcs’ followed by a lowercase letter are reserved
  for additional string and array functions. See Chapter 5 [String and Array Utilities],
  page 86.

- Names that end with ‘_t’ are reserved for additional type names.

In addition, some individual header files reserve names beyond those that they actually
define. You only need to worry about these restrictions if your program includes that
particular header file.

- The header file dirent.h reserves names prefixed with *d_*.
- The header file fcntl.h reserves names prefixed with *l_*, ‘F_’, ‘O_’, and ‘S_’.
- The header file grp.h reserves names prefixed with *gr_*.
- The header file limits.h reserves names suffixed with *_MAX*.
- The header file pwd.h reserves names prefixed with *pw_*.
- The header file signal.h reserves names prefixed with ‘sa_’ and ‘SA_’.
- The header file sys/stat.h reserves names prefixed with ‘st_’ and ‘S_’.
- The header file sys/times.h reserves names prefixed with ‘tms_’.
- The header file termios.h reserves names prefixed with ‘c_’, ‘V’, ‘I’, ‘O’, and ‘TC’; and
  names prefixed with ‘B’ followed by a digit.

Feature Test Macros
-------------------

The exact set of features available when you compile a source file is controlled by which
feature test macros you define.

If you compile your programs using ‘gcc -ansi’, you get only the ISO C library features,
unless you explicitly request additional features by defining one or more of the feature
macros. See Section “GNU CC Command Options” in The GNU CC Manual, for more
information about GCC options.

You should define these macros by using ‘#define’ preprocessor directives at the top of
your source code files. These directives must come before any #include of a system header
file. It is best to make them the very first thing in the file, preceded only by comments. You
could also use the ‘-D’ option to GCC, but it’s better if you make the source files indicate
their own meaning in a self-contained way.

This system exists to allow the library to conform to multiple standards. Although the
different standards are often described as supersets of each other, they are usually incompatible
because larger standards require functions with names that smaller ones reserve to
the user program. This is not mere pedantry — it has been a problem in practice. For
instance, some non-GNU programs define functions named getline that have nothing to
do with this library’s getline. They would not be compilable if all features were enabled
indiscriminately.

This should not be used to verify that a program conforms to a limited standard. It is
insufficient for this purpose, as it will not protect you from including header files outside
the standard, or relying on semantics undefined within the standard.

_POSIX_SOURCE
	If you define this macro, then the functionality from the POSIX.1 standard (IEEE
	Standard 1003.1) is available, as well as all of the ISO C facilities.
	The state of _POSIX_SOURCE is irrelevant if you define the macro _POSIX_C_SOURCE
	to a positive integer.

_POSIX_C_SOURCE 
	Define this macro to a positive integer to control which POSIX functionality is made
	available. The greater the value of this macro, the more functionality is made available.
	If you define this macro to a value greater than or equal to 1, then the functionality
	from the 1990 edition of the POSIX.1 standard (IEEE Standard 1003.1-1990) is made
	available.
	If you define this macro to a value greater than or equal to 2, then the functionality
	from the 1992 edition of the POSIX.2 standard (IEEE Standard 1003.2-1992) is made
	available.
	If you define this macro to a value greater than or equal to 199309L, then the functionality
	from the 1993 edition of the POSIX.1b standard (IEEE Standard 1003.1b-1993)
	is made available.
	Greater values for _POSIX_C_SOURCE will enable future extensions. The POSIX standards
	process will define these values as necessary, and the GNU C Library should support
	them some time after they become standardized. The 1996 edition of POSIX.1
	(ISO/IEC 9945-1: 1996) states that if you define _POSIX_C_SOURCE to a value greater
	than or equal to 199506L, then the functionality from the 1996 edition is made available.

_XOPEN_SOURCE
	Macro

_XOPEN_SOURCE_EXTENDED 
	If you define this macro, functionality described in the X/Open Portability Guide is
	included. This is a superset of the POSIX.1 and POSIX.2 functionality and in fact
	_POSIX_SOURCE and _POSIX_C_SOURCE are automatically defined.

	As the unification of all Unices, functionality only available in BSD and SVID is also
	included.
	If the macro _XOPEN_SOURCE_EXTENDED is also defined, even more functionality is
	available. The extra functions will make all functions available which are necessary
	for the X/Open Unix brand.
	If the macro _XOPEN_SOURCE has the value 500 this includes all functionality described
	so far plus some new definitions from the Single Unix Specification, version 2.

_LARGEFILE_SOURCE 
	If this macro is defined some extra functions are available which rectify a few shortcomings
	in all previous standards. Specifically, the functions fseeko and ftello are
	available. Without these functions the difference between the ISO C interface (fseek,
	ftell) and the low-level POSIX interface (lseek) would lead to problems.
	This macro was introduced as part of the Large File Support extension (LFS).

_LARGEFILE64_SOURCE
	If you define this macro an additional set of functions is made available which enables
	32 bit systems to use files of sizes beyond the usual limit of 2GB. This interface is
	not available if the system does not support files that large. On systems where the
	natural file size limit is greater than 2GB (i.e., on 64 bit systems) the new functions
	are identical to the replaced functions.
	The new functionality is made available by a new set of types and functions which
	replace the existing ones. The names of these new objects contain 64 to indicate the
	intention, e.g., off_t vs. off64_t and fseeko vs. fseeko64.
	This macro was introduced as part of the Large File Support extension (LFS). It is
	a transition interface for the period when 64 bit offsets are not generally used (see
	_FILE_OFFSET_BITS).

_FILE_OFFSET_BITS
	This macro determines which file system interface shall be used, one replacing the
	other. Whereas _LARGEFILE64_SOURCE makes the 64 bit interface available as an
	additional interface, _FILE_OFFSET_BITS allows the 64 bit interface to replace the
	old interface.
	If _FILE_OFFSET_BITS is undefined, or if it is defined to the value 32, nothing changes.
	The 32 bit interface is used and types like off_t have a size of 32 bits on 32 bit
	systems.
	If the macro is defined to the value 64, the large file interface replaces the old interface.
	I.e., the functions are not made available under different names (as they are
	with _LARGEFILE64_SOURCE). Instead the old function names now reference the new
	functions, e.g., a call to fseeko now indeed calls fseeko64.
	This macro should only be selected if the system provides mechanisms for handling
	large files. On 64 bit systems this macro has no effect since the *64 functions are
	identical to the normal functions.
	This macro was introduced as part of the Large File Support extension (LFS).

_ISOC99_SOURCE
	Until the revised ISO C standard is widely adopted the new features are not automatically
	enabled. The GNU C Library nevertheless has a complete implementation of
	the new standard and to enable the new features the macro _ISOC99_SOURCE should
	be defined.

_GNU_SOURCE
	If you define this macro, everything is included: ISO C89, ISO C99, POSIX.1,
	POSIX.2, BSD, SVID, X/Open, LFS, and GNU extensions. In the cases where
	POSIX.1 conflicts with BSD, the POSIX definitions take precedence.

_DEFAULT_SOURCE
	If you define this macro, most features are included apart from X/Open, LFS and
	GNU extensions: the effect is to enable features from the 2008 edition of POSIX,
	as well as certain BSD and SVID features without a separate feature test macro to
	control them. Defining this macro, on its own and without using compiler options such
	as -ansi or -std=c99, has the same effect as not defining any feature test macros;
	defining it together with other feature test macros, or when options such as -ansi
	are used, enables those features even when the other options would otherwise cause
	them to be disabled.

_REENTRANT
    Macro

_THREAD_SAFE
        If you define one of these macros, reentrant versions of several functions get declared.
        Some of the functions are specified in POSIX.1c but many others are only available
        on a few other systems or are unique to the GNU C Library. The problem is the
        delay in the standardization of the thread safe C library interface.
        Unlike on some other systems, no special version of the C library must be used for
        linking. There is only one version but while compiling this it must have been specified
        to compile as thread safe.

We recommend you use _GNU_SOURCE in new programs. If you don’t specify the ‘-ansi’
option to GCC, or other conformance options such as -std=c99, and don’t define any of
these macros explicitly, the effect is the same as defining _DEFAULT_SOURCE to 1.

When you define a feature test macro to request a larger class of features, it is harmless
to define in addition a feature test macro for a subset of those features. For example, if you
define _POSIX_C_SOURCE, then defining _POSIX_SOURCE as well has no effect. Likewise, if
you define _GNU_SOURCE, then defining either _POSIX_SOURCE or _POSIX_C_SOURCE as well
has no effect.

Roadmap to the Manual
======================

Here is an overview of the contents of the remaining chapters of this manual.

- Chapter 2 [Error Reporting], page 22, describes how errors detected by the library are
  reported.
- Chapter 3 [Virtual Memory Allocation And Paging], page 39, describes the GNU C
  Library’s facilities for managing and using virtual and real memory, including dynamic
  allocation of virtual memory. If you do not know in advance how much memory your
  program needs, you can allocate it dynamically instead, and manipulate it via pointers.
- Chapter 4 [Character Handling], page 76, contains information about character classification
  functions (such as isspace) and functions for performing case conversion.
- Chapter 5 [String and Array Utilities], page 86, has descriptions of functions for manipulating
  strings (null-terminated character arrays) and general byte arrays, including
  operations such as copying and comparison.
- Chapter 6 [Character Set Handling], page 127, contains information about manipulating
  characters and strings using character sets larger than will fit in the usual char data
  type.
- Chapter 7 [Locales and Internationalization], page 169, describes how selecting a particular
  country or language affects the behavior of the library. For example, the locale
  affects collation sequences for strings and how monetary values are formatted.
- Chapter 9 [Searching and Sorting], page 213, contains information about functions for
  searching and sorting arrays. You can use these functions on any kind of array by
  providing an appropriate comparison function.
- Chapter 10 [Pattern Matching], page 223, presents functions for matching regular expressions
  and shell file name patterns, and for expanding words as the shell does.
- Chapter 11 [Input/Output Overview], page 245, gives an overall look at the input and
  output facilities in the library, and contains information about basic concepts such as
  file names.
- Chapter 12 [Input/Output on Streams], page 250, describes I/O operations involving
  streams (or FILE * objects). These are the normal C library functions from stdio.h.
- Chapter 13 [Low-Level Input/Output], page 325, contains information about I/O operations
  on file descriptors. File descriptors are a lower-level mechanism specific to the
  Unix family of operating systems.
- Chapter 14 [File System Interface], page 379, has descriptions of operations on entire
  files, such as functions for deleting and renaming them and for creating new directories.
  This chapter also contains information about how you can access the attributes of a
  file, such as its owner and file protection modes.
- Chapter 15 [Pipes and FIFOs], page 426, contains information about simple interprocess
  communication mechanisms. Pipes allow communication between two related
  processes (such as between a parent and child), while FIFOs allow communication
  between processes sharing a common file system on the same machine.
- Chapter 16 [Sockets], page 431, describes a more complicated interprocess communication
  mechanism that allows processes running on different machines to communicate
  over a network. This chapter also contains information about Internet host addressing
  and how to use the system network databases.
- Chapter 17 [Low-Level Terminal Interface], page 479, describes how you can change
  the attributes of a terminal device. If you want to disable echo of characters typed by
  the user, for example, read this chapter.
- Chapter 19 [Mathematics], page 514, contains information about the math library functions.
  These include things like random-number generators and remainder functions on integers as well as the usual trigonometric and exponential functions on floating-point
  numbers.
- Chapter 20 [Low-Level Arithmetic Functions], page 562, describes functions for simple
  arithmetic, analysis of floating-point values, and reading numbers from strings.
- Chapter 21 [Date and Time], page 598, describes functions for measuring both calendar
  time and CPU time, as well as functions for setting alarms and timers.
- Chapter 23 [Non-Local Exits], page 655, contains descriptions of the setjmp and
  longjmp functions. These functions provide a facility for goto-like jumps which can
  jump from one function to another.
- Chapter 24 [Signal Handling], page 664, tells you all about signals—what they are, how
  to establish a handler that is called when a particular kind of signal is delivered, and
  how to prevent signals from arriving during critical sections of your program.
- Chapter 25 [The Basic Program/System Interface], page 708, tells how your programs
  can access their command-line arguments and environment variables.
- Chapter 26 [Processes], page 752, contains information about how to start new processes
  and run programs.
- Chapter 28 [Job Control], page 765, describes functions for manipulating process groups
  and the controlling terminal. This material is probably only of interest if you are writing
  a shell or other program which handles job control specially.
- Chapter 29 [System Databases and Name Service Switch], page 784, describes the services
  which are available for looking up names in the system databases, how to determine
  which service is used for which database, and how these services are implemented
  so that contributors can design their own services.
- Section 30.13 [User Database], page 813, and Section 30.14 [Group Database], page 817,
  tell you how to access the system user and group databases.
- Chapter 31 [System Management], page 824, describes functions for controlling and
  getting information about the hardware and software configuration your program is
  executing under.
- Chapter 32 [System Configuration Parameters], page 841, tells you how you can get
  information about various operating system limits. Most of these parameters are provided
  for compatibility with POSIX.
- Appendix A [C Language Facilities in the Library], page 881, contains information
  about library support for standard parts of the C language, including things like the
  sizeof operator and the symbolic constant NULL, how to write functions accepting
  variable numbers of arguments, and constants describing the ranges and other properties
  of the numerical types. There is also a simple debugging mechanism which allows
  you to put assertions in your code, and have diagnostic messages printed if the tests
  fail.
- Appendix B [Summary of Library Facilities], page 897, gives a summary of all the
  functions, variables, and macros in the library, with complete data types and function
  prototypes, and says what standard or system each is derived from.
- Appendix C [Installing the GNU C Library], page 1000, explains how to build and
  install the GNU C Library on your system, and how to report any bugs you might find.
- Appendix D [Library Maintenance], page 1008, explains how to add new functions or
  port the library to a new system.

If you already know the name of the facility you are interested in, you can look it up
in Appendix B [Summary of Library Facilities], page 897. This gives you a summary of its
syntax and a pointer to where you can find a more detailed description. This appendix is
particularly useful if you just want to verify the order and type of arguments to a function,
for example. It also tells you what standard or system each function, variable, or macro is
derived from.

