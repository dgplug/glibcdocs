===============================
System Configuration Parameters
===============================

The functions and macros listed in this chapter give information about configuration parameters
of the operating system—for example, capacity limits, presence of optional POSIX
features, and the default path for executable files (see Section 32.12 [String-Valued Parameters],
page 859).

General Capacity Limits
-----------------------

The POSIX.1 and POSIX.2 standards specify a number of parameters that describe capacity
limitations of the system. These limits can be fixed constants for a given operating
system, or they can vary from machine to machine. For example, some limit values may
be configurable by the system administrator, either at run time or by rebuilding the kernel,
and this should not require recompiling application programs.
Each of the following limit parameters has a macro that is defined in limits.h only if
the system has a fixed, uniform limit for the parameter in question. If the system allows
different file systems or files to have different limits, then the macro is undefined; use
sysconf to find out the limit that applies at a particular time on a particular machine. See
Section 32.4 [Using sysconf], page 844.
Each of these parameters also has another macro, with a name starting with ‘_POSIX’,
which gives the lowest value that the limit is allowed to have on any POSIX system. See
Section 32.5 [Minimum Values for General Capacity Limits], page 852.

``int ARG_MAX`` [Macro]
        If defined, the unvarying maximum combined length of the argv and environ arguments
that can be passed to the exec functions.

``int CHILD_MAX`` [Macro]
        If defined, the unvarying maximum number of processes that can exist with the same
real user ID at any one time. In BSD and GNU, this is controlled by the RLIMIT_
NPROC resource limit; see Section 22.2 [Limiting Resource Usage], page 635.

``int OPEN_MAX`` [Macro]
        If defined, the unvarying maximum number of files that a single process can have open
simultaneously. In BSD and GNU, this is controlled by the RLIMIT_NOFILE resource
limit; see Section 22.2 [Limiting Resource Usage], page 635.

``int STREAM_MAX`` [Macro]
        If defined, the unvarying maximum number of streams that a single process can have
open simultaneously. See Section 12.3 [Opening Streams], page 251.

``int TZNAME_MAX`` [Macro]
        If defined, the unvarying maximum length of a time zone name. See Section 21.4.8
[Functions and Variables for Time Zones], page 627.
These limit macros are always defined in limits.h

``int NGROUPS_MAX`` [Macro]
        The maximum number of supplementary group IDs that one process can have.
The value of this macro is actually a lower bound for the maximum. That is, you can
count on being able to have that many supplementary group IDs, but a particular
machine might let you have even more. You can use sysconf to see whether a
particular machine will let you have more (see Section 32.4 [Using sysconf], page 844).

``ssize_t SSIZE_MAX`` [Macro]
        The largest value that can fit in an object of type ssize_t. Effectively, this is the
limit on the number of bytes that can be read or written in a single operation.
This macro is defined in all POSIX systems because this limit is never configurable.

``int RE_DUP_MAX`` [Macro]
        The largest number of repetitions you are guaranteed is allowed in the construct
‘\{min,max\}’ in a regular expression.
The value of this macro is actually a lower bound for the maximum. That is, you can
count on being able to have that many repetitions, but a particular machine might
let you have even more. You can use sysconf to see whether a particular machine
will let you have more (see Section 32.4 [Using sysconf], page 844). And even the
value that sysconf tells you is just a lower bound—larger values might work.
This macro is defined in all POSIX.2 systems, because POSIX.2 says it should always
be defined even if there is no specific imposed limit.

Overall System Options
----------------------

POSIX defines certain system-specific options that not all POSIX systems support. Since
these options are provided in the kernel, not in the library, simply using the GNU C Library
does not guarantee any of these features is supported; it depends on the system you are
using.
You can test for the availability of a given option using the macros in this section,
together with the function sysconf. The macros are defined only if you include unistd.h.
For the following macros, if the macro is defined in unistd.h, then the option is supported.
Otherwise, the option may or may not be supported; use sysconf to find out. See
Section 32.4 [Using sysconf], page 844.

``int _POSIX_JOB_CONTROL`` [Macro]
        If this symbol is defined, it indicates that the system supports job control. Otherwise,
the implementation behaves as if all processes within a session belong to a single
process group. See Chapter 28 [Job Control], page 765.

``int _POSIX_SAVED_IDS`` [Macro]
        If this symbol is defined, it indicates that the system remembers the effective user and
group IDs of a process before it executes an executable file with the set-user-ID or setgroup-ID
bits set, and that explicitly changing the effective user or group IDs back to
these values is permitted. If this option is not defined, then if a nonprivileged process
changes its effective user or group ID to the real user or group ID of the process, it
can’t change it back again. See Section 30.8 [Enabling and Disabling Setuid Access],
page 800.

For the following macros, if the macro is defined in unistd.h, then its value indicates
whether the option is supported. A value of -1 means no, and any other value means yes.
If the macro is not defined, then the option may or may not be supported; use sysconf to
find out. See Section 32.4 [Using sysconf], page 844.

``int _POSIX2_C_DEV`` [Macro]
        If this symbol is defined, it indicates that the system has the POSIX.2 C compiler
command, c89. The GNU C Library always defines this as 1, on the assumption that
you would not have installed it if you didn’t have a C compiler.

``int _POSIX2_FORT_DEV`` [Macro]
        If this symbol is defined, it indicates that the system has the POSIX.2 Fortran compiler
command, fort77. The GNU C Library never defines this, because we don’t
know what the system has.

``int _POSIX2_FORT_RUN`` [Macro]
        If this symbol is defined, it indicates that the system has the POSIX.2 asa command
to interpret Fortran carriage control. The GNU C Library never defines this, because
we don’t know what the system has.

``int _POSIX2_LOCALEDEF`` [Macro]
        If this symbol is defined, it indicates that the system has the POSIX.2 localedef
command. The GNU C Library never defines this, because we don’t know what the
system has.

``int _POSIX2_SW_DEV`` [Macro]
        If this symbol is defined, it indicates that the system has the POSIX.2 commands ar,
make, and strip. The GNU C Library always defines this as 1, on the assumption
that you had to have ar and make to install the library, and it’s unlikely that strip
would be absent when those are present.

Which Version of POSIX is Supported
-----------------------------------

``long int _POSIX_VERSION`` [Macro]
        This constant represents the version of the POSIX.1 standard to which the implementation
conforms. For an implementation conforming to the 1995 POSIX.1 standard,
the value is the integer 199506L.

``_POSIX_VERSION`` is always defined (in unistd.h) in any POSIX system.
Usage Note: Don’t try to test whether the system supports POSIX by including
unistd.h and then checking whether _POSIX_VERSION is defined. On a non-POSIX
system, this will probably fail because there is no unistd.h. We do not know of any
way you can reliably test at compilation time whether your target system supports
POSIX or whether unistd.h exists.

``long int _POSIX2_C_VERSION`` [Macro]
        This constant represents the version of the POSIX.2 standard which the library and
system kernel support. We don’t know what value this will be for the first version of
the POSIX.2 standard, because the value is based on the year and month in which
the standard is officially adopted.

The value of this symbol says nothing about the utilities installed on the system.
Usage Note: You can use this macro to tell whether a POSIX.1 system library supports
POSIX.2 as well. Any POSIX.1 system contains unistd.h, so include that file
and then test defined (_POSIX2_C_VERSION).

Using sysconf
-------------

When your system has configurable system limits, you can use the sysconf function to
find out the value that applies to any particular machine. The function and the associated
parameter constants are declared in the header file unistd.h.

Definition of sysconf
---------------------

long int sysconf (int parameter) [Function]

Preliminary: | MT-Safe env | AS-Unsafe lock heap | AC-Unsafe lock mem fd | See
Section 1.2.2.1 [POSIX Safety Concepts], page 2.
This function is used to inquire about runtime system parameters. The parameter
argument should be one of the ‘_SC_’ symbols listed below.
The normal return value from sysconf is the value you requested. A value of -1 is
returned both if the implementation does not impose a limit, and in case of an error.
The following errno error conditions are defined for this function:
EINVAL The value of the parameter is invalid.

Constants for sysconf Parameters
--------------------------------

Here are the symbolic constants for use as the parameter argument to sysconf. The values
are all integer constants (more specifically, enumeration type values).

``_SC_ARG_MAX``
        Inquire about the parameter corresponding to ARG_MAX.

``_SC_CHILD_MAX``
        Inquire about the parameter corresponding to CHILD_MAX.

``_SC_OPEN_MAX``
        Inquire about the parameter corresponding to OPEN_MAX.

``_SC_STREAM_MAX``
        Inquire about the parameter corresponding to STREAM_MAX.

``_SC_TZNAME_MAX``
        Inquire about the parameter corresponding to TZNAME_MAX.

``_SC_NGROUPS_MAX``
        Inquire about the parameter corresponding to NGROUPS_MAX.

``_SC_JOB_CONTROL``
        Inquire about the parameter corresponding to _POSIX_JOB_CONTROL.

``_SC_SAVED_IDS``
        Inquire about the parameter corresponding to _POSIX_SAVED_IDS.

``_SC_VERSION``
        Inquire about the parameter corresponding to _POSIX_VERSION.

``_SC_CLK_TCK``
        Inquire about the number of clock ticks per second; see Section 21.3.1 [CPU
Time Inquiry], page 600. The corresponding parameter CLK_TCK is obsolete.

``_SC_CHARCLASS_NAME_MAX``
        Inquire about the parameter corresponding to maximal length allowed for a
character class name in an extended locale specification. These extensions are
not yet standardized and so this option is not standardized as well.

``_SC_REALTIME_SIGNALS``
        Inquire about the parameter corresponding to _POSIX_REALTIME_SIGNALS.

``_SC_PRIORITY_SCHEDULING``
        Inquire about the parameter corresponding to _POSIX_PRIORITY_SCHEDULING.

``_SC_TIMERS``
        Inquire about the parameter corresponding to _POSIX_TIMERS.

``_SC_ASYNCHRONOUS_IO``
        Inquire about the parameter corresponding to _POSIX_ASYNCHRONOUS_IO.

``_SC_PRIORITIZED_IO``
        Inquire about the parameter corresponding to _POSIX_PRIORITIZED_IO.

``_SC_SYNCHRONIZED_IO``
        Inquire about the parameter corresponding to _POSIX_SYNCHRONIZED_IO.

``_SC_FSYNC``
        Inquire about the parameter corresponding to _POSIX_FSYNC.

``_SC_MAPPED_FILES``
        Inquire about the parameter corresponding to _POSIX_MAPPED_FILES.

``_SC_MEMLOCK``
        Inquire about the parameter corresponding to _POSIX_MEMLOCK.

``_SC_MEMLOCK_RANGE``
        Inquire about the parameter corresponding to _POSIX_MEMLOCK_RANGE.

``_SC_MEMORY_PROTECTION``
        Inquire about the parameter corresponding to _POSIX_MEMORY_PROTECTION.

``_SC_MESSAGE_PASSING``
        Inquire about the parameter corresponding to _POSIX_MESSAGE_PASSING.

``_SC_SEMAPHORES``
        Inquire about the parameter corresponding to _POSIX_SEMAPHORES.

``_SC_SHARED_MEMORY_OBJECTS``
        Inquire about the parameter corresponding to _POSIX_SHARED_MEMORY_OBJECTS.

``_SC_AIO_LISTIO_MAX``
        Inquire about the parameter corresponding to _POSIX_AIO_LISTIO_MAX.

``_SC_AIO_MAX``
        Inquire about the parameter corresponding to _POSIX_AIO_MAX.

``_SC_AIO_PRIO_DELTA_MAX``
        Inquire the value by which a process can decrease its asynchronous I/O priority
level from its own scheduling priority. This corresponds to the run-time
invariant value AIO_PRIO_DELTA_MAX.

``_SC_DELAYTIMER_MAX``
        Inquire about the parameter corresponding to _POSIX_DELAYTIMER_MAX.

``_SC_MQ_OPEN_MAX``
        Inquire about the parameter corresponding to _POSIX_MQ_OPEN_MAX.

``_SC_MQ_PRIO_MAX``
        Inquire about the parameter corresponding to _POSIX_MQ_PRIO_MAX.

``_SC_RTSIG_MAX``
        Inquire about the parameter corresponding to _POSIX_RTSIG_MAX.

``_SC_SEM_NSEMS_MAX``
        Inquire about the parameter corresponding to _POSIX_SEM_NSEMS_MAX.

``_SC_SEM_VALUE_MAX``
        Inquire about the parameter corresponding to _POSIX_SEM_VALUE_MAX.

``_SC_SIGQUEUE_MAX``
        Inquire about the parameter corresponding to _POSIX_SIGQUEUE_MAX.

``_SC_TIMER_MAX``
        Inquire about the parameter corresponding to _POSIX_TIMER_MAX.

``_SC_PII``
        Inquire about the parameter corresponding to _POSIX_PII.

``_SC_PII_XTI``
        Inquire about the parameter corresponding to _POSIX_PII_XTI.

``_SC_PII_SOCKET``
        Inquire about the parameter corresponding to _POSIX_PII_SOCKET.

``_SC_PII_INTERNET``
        Inquire about the parameter corresponding to _POSIX_PII_INTERNET.

``_SC_PII_OSI``
        Inquire about the parameter corresponding to _POSIX_PII_OSI.

``_SC_SELECT``
        Inquire about the parameter corresponding to _POSIX_SELECT.

``_SC_UIO_MAXIOV``
        Inquire about the parameter corresponding to _POSIX_UIO_MAXIOV.

``_SC_PII_INTERNET_STREAM``
        Inquire about the parameter corresponding to _POSIX_PII_INTERNET_STREAM.

``_SC_PII_INTERNET_DGRAM``
        Inquire about the parameter corresponding to _POSIX_PII_INTERNET_DGRAM.

``_SC_PII_OSI_COTS``
        Inquire about the parameter corresponding to _POSIX_PII_OSI_COTS.

``_SC_PII_OSI_CLTS``
        Inquire about the parameter corresponding to _POSIX_PII_OSI_CLTS.

``_SC_PII_OSI_M``
        Inquire about the parameter corresponding to _POSIX_PII_OSI_M.

``_SC_T_IOV_MAX``
        Inquire the value of the value associated with the T_IOV_MAX variable.

``_SC_THREADS``
        Inquire about the parameter corresponding to _POSIX_THREADS.

``_SC_THREAD_SAFE_FUNCTIONS``
        Inquire about the parameter corresponding to
_POSIX_THREAD_SAFE_FUNCTIONS.

``_SC_GETGR_R_SIZE_MAX``
        Inquire about the parameter corresponding to _POSIX_GETGR_R_SIZE_MAX.

``_SC_GETPW_R_SIZE_MAX``
        Inquire about the parameter corresponding to _POSIX_GETPW_R_SIZE_MAX.

``_SC_LOGIN_NAME_MAX``
        Inquire about the parameter corresponding to _POSIX_LOGIN_NAME_MAX.

``_SC_TTY_NAME_MAX``
        Inquire about the parameter corresponding to _POSIX_TTY_NAME_MAX.

``_SC_THREAD_DESTRUCTOR_ITERATIONS``
        Inquire about the parameter corresponding to _POSIX_THREAD_DESTRUCTOR_
ITERATIONS.

``_SC_THREAD_KEYS_MAX``
        Inquire about the parameter corresponding to _POSIX_THREAD_KEYS_MAX.

``_SC_THREAD_STACK_MIN``
        Inquire about the parameter corresponding to _POSIX_THREAD_STACK_MIN.

``_SC_THREAD_THREADS_MAX``
        Inquire about the parameter corresponding to _POSIX_THREAD_THREADS_MAX.

``_SC_THREAD_ATTR_STACKADDR``
        Inquire about the parameter corresponding to a _POSIX_THREAD_ATTR_STACKADDR.

``_SC_THREAD_ATTR_STACKSIZE``
        Inquire about the parameter corresponding to
_POSIX_THREAD_ATTR_STACKSIZE.

``_SC_THREAD_PRIORITY_SCHEDULING``
        Inquire about the parameter corresponding to _POSIX_THREAD_PRIORITY_
SCHEDULING.

``_SC_THREAD_PRIO_INHERIT``
        Inquire about the parameter corresponding to _POSIX_THREAD_PRIO_INHERIT.

``_SC_THREAD_PRIO_PROTECT``
        Inquire about the parameter corresponding to _POSIX_THREAD_PRIO_PROTECT.

``_SC_THREAD_PROCESS_SHARED``
        Inquire about the parameter corresponding to _POSIX_THREAD_PROCESS_
SHARED.

``_SC_2_C_DEV``
        Inquire about whether the system has the POSIX.2 C compiler command, c89.

``_SC_2_FORT_DEV``
        Inquire about whether the system has the POSIX.2 Fortran compiler command,
fort77.

``_SC_2_FORT_RUN``
        Inquire about whether the system has the POSIX.2 asa command to interpret
Fortran carriage control.

``_SC_2_LOCALEDEF``
        Inquire about whether the system has the POSIX.2 localedef command.

``_SC_2_SW_DEV``
        Inquire about whether the system has the POSIX.2 commands ar, make, and
strip.

``_SC_BC_BASE_MAX``
        Inquire about the maximum value of obase in the bc utility.

``_SC_BC_DIM_MAX``
        Inquire about the maximum size of an array in the bc utility.

``_SC_BC_SCALE_MAX``
        Inquire about the maximum value of scale in the bc utility.

``_SC_BC_STRING_MAX``
        Inquire about the maximum size of a string constant in the bc utility.

``_SC_COLL_WEIGHTS_MAX``
        Inquire about the maximum number of weights that can necessarily be used in
defining the collating sequence for a locale.

``_SC_EXPR_NEST_MAX``
        Inquire about the maximum number of expressions nested within parentheses
when using the expr utility.

``_SC_LINE_MAX``
        Inquire about the maximum size of a text line that the POSIX.2 text utilities
can handle.

``_SC_EQUIV_CLASS_MAX``
        Inquire about the maximum number of weights that can be assigned to an entry
of the LC_COLLATE category ‘order’ keyword in a locale definition. The GNU
C Library does not presently support locale definitions.

``SC_EQUIV_CLASS_MAX``
        Inquire about the maximum number of weights that can be assigned to an entry
of the LC_COLLATE category ‘order’ keyword in a locale definition. The GNU
C Library does not presently support locale definitions.

``_SC_VERSION``
        Inquire about the version number of POSIX.1 that the library and kernel support.

``_SC_2_VERSION``
        Inquire about the version number of POSIX.2 that the system utilities support.

``_SC_PAGESIZE``
        Inquire about the virtual memory page size of the machine. getpagesize
returns the same value (see Section 22.4.2 [How to get information about the
memory subsystem?], page 651).

``_SC_NPROCESSORS_CONF``
        Inquire about the number of configured processors.

``_SC_NPROCESSORS_ONLN``
        Inquire about the number of processors online.

``_SC_PHYS_PAGES``
        Inquire about the number of physical pages in the system.

``_SC_AVPHYS_PAGES``
        Inquire about the number of available physical pages in the system.

``_SC_ATEXIT_MAX``
        Inquire about the number of functions which can be registered as termination
functions for atexit; see Section 25.7.3 [Cleanups on Exit], page 749.

``_SC_XOPEN_VERSION``
        Inquire about the parameter corresponding to _XOPEN_VERSION.

``_SC_XOPEN_XCU_VERSION``
        Inquire about the parameter corresponding to _XOPEN_XCU_VERSION.

``_SC_XOPEN_UNIX``
        Inquire about the parameter corresponding to _XOPEN_UNIX.

``_SC_XOPEN_REALTIME``
        Inquire about the parameter corresponding to _XOPEN_REALTIME.

``_SC_XOPEN_REALTIME_THREADS``
        Inquire about the parameter corresponding to _XOPEN_REALTIME_THREADS.

``_SC_XOPEN_LEGACY``
        Inquire about the parameter corresponding to _XOPEN_LEGACY.

``_SC_XOPEN_CRYPT``
        Inquire about the parameter corresponding to _XOPEN_CRYPT.

``_SC_XOPEN_ENH_I18N``
        Inquire about the parameter corresponding to _XOPEN_ENH_I18N.

``_SC_XOPEN_SHM``
        Inquire about the parameter corresponding to _XOPEN_SHM.

``_SC_XOPEN_XPG2``
        Inquire about the parameter corresponding to _XOPEN_XPG2.

``_SC_XOPEN_XPG3``
        Inquire about the parameter corresponding to _XOPEN_XPG3.

``_SC_XOPEN_XPG4``
        Inquire about the parameter corresponding to _XOPEN_XPG4.

``_SC_CHAR_BIT``
        Inquire about the number of bits in a variable of type char.

``_SC_CHAR_MAX``
        Inquire about the maximum value which can be stored in a variable of type
char.

``_SC_CHAR_MIN``
    Inquire about the minimum value which can be stored in a variable of type
char.

``_SC_INT_MAX``
    Inquire about the maximum value which can be stored in a variable of type
int.

``_SC_INT_MIN``
        Inquire about the minimum value which can be stored in a variable of type int.

``_SC_LONG_BIT``
        Inquire about the number of bits in a variable of type long int.

``_SC_WORD_BIT``
        Inquire about the number of bits in a variable of a register word.

``_SC_MB_LEN_MAX``
        Inquire the maximum length of a multi-byte representation of a wide character
value.

``_SC_NZERO``
        Inquire about the value used to internally represent the zero priority level for
the process execution.

``SC_SSIZE_MAX``
        Inquire about the maximum value which can be stored in a variable of type
ssize_t.

``_SC_SCHAR_MAX``
        Inquire about the maximum value which can be stored in a variable of type
signed char.

``_SC_SCHAR_MIN``
        Inquire about the minimum value which can be stored in a variable of type
signed char.

``_SC_SHRT_MAX``
        Inquire about the maximum value which can be stored in a variable of type
short int.

``_SC_SHRT_MIN``
        Inquire about the minimum value which can be stored in a variable of type
short int.

``_SC_UCHAR_MAX``
        Inquire about the maximum value which can be stored in a variable of type
unsigned char.

``_SC_UINT_MAX``
        Inquire about the maximum value which can be stored in a variable of type
unsigned int.

``_SC_ULONG_MAX``
        Inquire about the maximum value which can be stored in a variable of type
unsigned long int.

``_SC_USHRT_MAX``
        Inquire about the maximum value which can be stored in a variable of type
unsigned short int.

``_SC_NL_ARGMAX``
        Inquire about the parameter corresponding to NL_ARGMAX.

``_SC_NL_LANGMAX``
        Inquire about the parameter corresponding to NL_LANGMAX.

``_SC_NL_MSGMAX``
        Inquire about the parameter corresponding to NL_MSGMAX.

``_SC_NL_NMAX``
        Inquire about the parameter corresponding to NL_NMAX.

``_SC_NL_SETMAX``
        Inquire about the parameter corresponding to NL_SETMAX.

``_SC_NL_TEXTMAX``
        Inquire about the parameter corresponding to NL_TEXTMAX.

Examples of sysconf
-------------------

We recommend that you first test for a macro definition for the parameter you are interested
in, and call sysconf only if the macro is not defined. For example, here is how to test
whether job control is supported:

.. code-block:: c
   int
   have_job_control (void)
   {
   #ifdef _POSIX_JOB_CONTROL
       return 1;
   #else
      int value = sysconf (_SC_JOB_CONTROL);
      if (value < 0)
        /* If the system is that badly wedged,
        therefls no use trying to go on.*/
        fatal (strerror (errno));
        return value;
   #endif
   }

Here is how to get the value of a numeric limit:

.. code-block:: c

  int
  get_child_max ()
  {
  #ifdef CHILD_MAX
      return CHILD_MAX;
  #else
      int value = sysconf (_SC_CHILD_MAX);
      if (value < 0)
          fatal (strerror (errno));
          return value;
  #endif
  }

Minimum Values for General Capacity Limits
------------------------------------------

Here are the names for the POSIX minimum upper bounds for the system limit parameters.
The significance of these values is that you can safely push to these limits without checking
whether the particular system you are using can go that far.

``_POSIX_AIO_LISTIO_MAX``
        The most restrictive limit permitted by POSIX for the maximum number of I/O
operations that can be specified in a list I/O call. The value of this constant is
2; thus you can add up to two new entries of the list of outstanding operations.

``_POSIX_AIO_MAX``
        The most restrictive limit permitted by POSIX for the maximum number of
outstanding asynchronous I/O operations. The value of this constant is 1. So
you cannot expect that you can issue more than one operation and immediately
continue with the normal work, receiving the notifications asynchronously.

``_POSIX_ARG_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum combined length of the argv and environ arguments that can be
passed to the exec functions. Its value is 4096.

``_POSIX_CHILD_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum number of simultaneous processes per real user ID. Its value is
6.

``_POSIX_NGROUPS_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum number of supplementary group IDs per process. Its value is 0.

``_POSIX_OPEN_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for the
maximum number of files that a single process can have open simultaneously.
Its value is 16.

``_POSIX_SSIZE_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum value that can be stored in an object of type ssize_t. Its value
is 32767.

``_POSIX_STREAM_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum number of streams that a single process can have open simultaneously.
Its value is 8.

``_POSIX_TZNAME_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the maximum length of a time zone name. Its value is 3.

``_POSIX2_RE_DUP_MAX``
        The value of this macro is the most restrictive limit permitted by POSIX for
the numbers used in the ‘\{min,max\}’ construct in a regular expression. Its
value is 255.

Limits on File System Capacity
------------------------------

The POSIX.1 standard specifies a number of parameters that describe the limitations of
the file system. It’s possible for the system to have a fixed, uniform limit for a parameter,
but this isn’t the usual case. On most systems, it’s possible for different file systems (and,
for some parameters, even different files) to have different maximum limits. For example,
this is very likely if you use NFS to mount some of the file systems from other machines.
Each of the following macros is defined in limits.h only if the system has a fixed,
uniform limit for the parameter in question. If the system allows different file systems or
files to have different limits, then the macro is undefined; use pathconf or fpathconf to find
out the limit that applies to a particular file. See Section 32.9 [Using pathconf], page 856.
Each parameter also has another macro, with a name starting with ‘_POSIX’, which gives
the lowest value that the limit is allowed to have on any POSIX system. See Section 32.8
[Minimum Values for File System Limits], page 855.

``int LINK_MAX`` [Macro]
        The uniform system limit (if any) for the number of names for a given file. See
Section 14.4 [Hard Links], page 394.

``int MAX_CANON`` [Macro]
        The uniform system limit (if any) for the amount of text in a line of input when
input editing is enabled. See Section 17.3 [Two Styles of Input: Canonical or Not],
page 480.

``int MAX_INPUT`` [Macro]
        The uniform system limit (if any) for the total number of characters typed ahead as
input. See Section 17.2 [I/O Queues], page 480.

``int NAME_MAX`` [Macro]
        The uniform system limit (if any) for the length of a file name component, not including
the terminating null character.
Portability Note: On some systems, the GNU C Library defines NAME_MAX, but does
not actually enforce this limit.

``int PATH_MAX`` [Macro]
        The uniform system limit (if any) for the length of an entire file name (that is, the
argument given to system calls such as open), including the terminating null character.
Portability Note: The GNU C Library does not enforce this limit even if PATH_MAX
is defined.

``int PIPE_BUF`` [Macro]
        The uniform system limit (if any) for the number of bytes that can be written atomically
to a pipe. If multiple processes are writing to the same pipe simultaneously, output
from different processes might be interleaved in chunks of this size. See Chapter 15
[Pipes and FIFOs], page 426.
These are alternative macro names for some of the same information.

``int MAXNAMLEN`` [Macro]
        This is the BSD name for NAME_MAX. It is defined in dirent.h.

``int FILENAME_MAX`` [Macro]
        The value of this macro is an integer constant expression that represents the maximum
length of a file name string. It is defined in stdio.h.
Unlike PATH_MAX, this macro is defined even if there is no actual limit imposed. In
such a case, its value is typically a very large number. This is always the case on
GNU/Hurd systems.
Usage Note: Don’t use FILENAME_MAX as the size of an array in which to store a
file name! You can’t possibly make an array that big! Use dynamic allocation (see
Section 3.2 [Allocating Storage For Program Data], page 40) instead.

Optional Features in File Support
---------------------------------

POSIX defines certain system-specific options in the system calls for operating on files.
Some systems support these options and others do not. Since these options are provided
in the kernel, not in the library, simply using the GNU C Library does not guarantee that
any of these features is supported; it depends on the system you are using. They can also
vary between file systems on a single machine.
This section describes the macros you can test to determine whether a particular option
is supported on your machine. If a given macro is defined in unistd.h, then its value says
whether the corresponding feature is supported. (A value of -1 indicates no; any other
value indicates yes.) If the macro is undefined, it means particular files may or may not
support the feature.
Since all the machines that support the GNU C Library also support NFS, one can
never make a general statement about whether all file systems support the _POSIX_CHOWN_
RESTRICTED and _POSIX_NO_TRUNC features. So these names are never defined as macros
in the GNU C Library.

``int _POSIX_CHOWN_RESTRICTED [Macro]``
        If this option is in effect, the chown function is restricted so that the only changes
        permitted to nonprivileged processes is to change the group owner of a file to either
        be the effective group ID of the process, or one of its supplementary group IDs. See
        Section 14.9.4 [File Owner], page 408.

``int _POSIX_NO_TRUNC [Macro]``
        If this option is in effect, file name components longer than NAME_MAX generate an
        ENAMETOOLONG error. Otherwise, file name components that are too long are silently
        truncated.

``unsigned char _POSIX_VDISABLE [Macro]``
    This option is only meaningful for files that are terminal devices. If it is enabled, then
    handling for special control characters can be disabled individually. See Section 17.4.9
    [Special Characters], page 492.
    If one of these macros is undefined, that means that the option might be in effect for
    some files and not for others. To inquire about a particular file, call pathconf or fpathconf.
    See Section 32.9 [Using pathconf], page 856.

Minimum Values for File System Limits
-------------------------------------

Here are the names for the POSIX minimum upper bounds for some of the above parameters.
The significance of these values is that you can safely push to these limits without checking
whether the particular system you are using can go that far. In most cases GNU systems
do not have these strict limitations. The actual limit should be requested if necessary.

``_POSIX_LINK_MAX``
        The most restrictive limit permitted by POSIX for the maximum value of a
        file’s link count. The value of this constant is 8; thus, you can always make up
        to eight names for a file without running into a system limit.

``_POSIX_MAX_CANON``
        The most restrictive limit permitted by POSIX for the maximum number of
        bytes in a canonical input line from a terminal device. The value of this constant
        is 255.

``_POSIX_MAX_INPUT``
        The most restrictive limit permitted by POSIX for the maximum number of
        bytes in a terminal device input queue (or typeahead buffer). See Section 17.4.4
        [Input Modes], page 484. The value of this constant is 255.

``_POSIX_NAME_MAX``
        The most restrictive limit permitted by POSIX for the maximum number of
        bytes in a file name component. The value of this constant is 14.

``_POSIX_PATH_MAX``
        The most restrictive limit permitted by POSIX for the maximum number of
        bytes in a file name. The value of this constant is 256.

``_POSIX_PIPE_BUF``
        The most restrictive limit permitted by POSIX for the maximum number of
        bytes that can be written atomically to a pipe. The value of this constant is
        512.

``SYMLINK_MAX``
        Maximum number of bytes in a symbolic link.

``POSIX_REC_INCR_XFER_SIZE``
        Recommended increment for file transfer sizes between the POSIX_REC_MIN_XFER_SIZE and POSIX_REC_MAX_XFER_SIZE values.

``POSIX_REC_MAX_XFER_SIZE``
        Maximum recommended file transfer size.

``POSIX_REC_MIN_XFER_SIZE``
        Minimum recommended file transfer size.

``POSIX_REC_XFER_ALIGN``
        Recommended file transfer buffer alignment.

Using pathconf
--------------

When your machine allows different files to have different values for a file system parameter,
you can use the functions in this section to find out the value that applies to any particular
file.
These functions and the associated constants for the parameter argument are declared
in the header file unistd.h.

``long int pathconf (const char filename, int parameter) [Function]``

        Preliminary: | MT-Safe | AS-Unsafe lock heap | AC-Unsafe lock fd mem | See
Section 1.2.2.1 [POSIX Safety Concepts], page 2.
This function is used to inquire about the limits that apply to the file named filename.
The parameter argument should be one of the ‘_PC_’ constants listed below.
The normal return value from pathconf is the value you requested. A value of -1 is
returned both if the implementation does not impose a limit, and in case of an error.
In the former case, errno is not set, while in the latter case, errno is set to indicate
the cause of the problem. So the only way to use this function robustly is to store 0
into errno just before calling it.
Besides the usual file name errors (see Section 11.2.3 [File Name Errors], page 248),
the following error condition is defined for this function:

EINVAL The value of parameter is invalid, or the implementation doesn’t support
the parameter for the specific file.

``long int fpathconf (int filedes, int parameter) [Function]``
        Preliminary: | MT-Safe | AS-Unsafe lock heap | AC-Unsafe lock fd mem | See
Section 1.2.2.1 [POSIX Safety Concepts], page 2.

This is just like pathconf except that an open file descriptor is used to specify the
file for which information is requested, instead of a file name.
The following errno error conditions are defined for this function:
EBADF The filedes argument is not a valid file descriptor.
EINVAL The value of parameter is invalid, or the implementation doesn’t support
the parameter for the specific file

Here are the symbolic constants that you can use as the parameter argument to pathconf
and fpathconf. The values are all integer constants.

``_PC_LINK_MAX``
        Inquire about the value of LINK_MAX.

``_PC_MAX_CANON``
        Inquire about the value of MAX_CANON.

``_PC_MAX_INPUT``
        Inquire about the value of MAX_INPUT.

``_PC_NAME_MAX``
        Inquire about the value of NAME_MAX.

``_PC_PATH_MAX``
        Inquire about the value of PATH_MAX.

``_PC_PIPE_BUF``
        Inquire about the value of PIPE_BUF.

``_PC_CHOWN_RESTRICTED``
        Inquire about the value of _POSIX_CHOWN_RESTRICTED.

``_PC_NO_TRUNC``
        Inquire about the value of _POSIX_NO_TRUNC.

``_PC_VDISABLE``
        Inquire about the value of _POSIX_VDISABLE.

``_PC_SYNC_IO``
        Inquire about the value of _POSIX_SYNC_IO.

``_PC_ASYNC_IO``
        Inquire about the value of _POSIX_ASYNC_IO.

``_PC_PRIO_IO``
        Inquire about the value of _POSIX_PRIO_IO.

``_PC_FILESIZEBITS``
        Inquire about the availability of large files on the filesystem.

``_PC_REC_INCR_XFER_SIZE``
        Inquire about the value of POSIX_REC_INCR_XFER_SIZE.

``_PC_REC_MAX_XFER_SIZE``
        Inquire about the value of POSIX_REC_MAX_XFER_SIZE.

``_PC_REC_MIN_XFER_SIZE``
        Inquire about the value of POSIX_REC_MIN_XFER_SIZE.

``_PC_REC_XFER_ALIGN``
        Inquire about the value of POSIX_REC_XFER_ALIGN.

Portability Note: On some systems, the GNU C Library does not enforce ``_PC_NAME_MAX``
or ``_PC_PATH_MAX`` limits.

Utility Program Capacity Limits
-------------------------------

The POSIX.2 standard specifies certain system limits that you can access through sysconf
that apply to utility behavior rather than the behavior of the library or the operating
system.
The GNU C Library defines macros for these limits, and sysconf returns values for
them if you ask; but these values convey no meaningful information. They are simply the
smallest values that POSIX.2 permits.

``int BC_BASE_MAX [Macro]``
        The largest value of obase that the bc utility is guaranteed to support.

``int BC_DIM_MAX [Macro]``
        The largest number of elements in one array that the bc utility is guaranteed to
support.

``int BC_SCALE_MAX [Macro]``
        The largest value of scale that the bc utility is guaranteed to support.

``int BC_STRING_MAX [Macro]``
        The largest number of characters in one string constant that the bc utility is guaranteed
        to support.

``int COLL_WEIGHTS_MAX [Macro]``
        The largest number of weights that can necessarily be used in defining the collating
        sequence for a locale.

``int EXPR_NEST_MAX [Macro]``
        The maximum number of expressions that can be nested within parenthesis by the
        expr utility.

``int LINE_MAX [Macro]``
        The largest text line that the text-oriented POSIX.2 utilities can support. (If you are
        using the GNU versions of these utilities, then there is no actual limit except that
        imposed by the available virtual memory, but there is no way that the library can tell
        you this.)

``int EQUIV_CLASS_MAX [Macro]``
        The maximum number of weights that can be assigned to an entry of the LC_COLLATE
        category ‘order’ keyword in a locale definition. The GNU C Library does not
        presently support locale definitions.

Minimum Values for Utility Limits
---------------------------------

``_POSIX2_BC_BASE_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum value of
        obase in the bc utility. Its value is 99.

``_POSIX2_BC_DIM_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum size of an
        array in the bc utility. Its value is 2048.

``_POSIX2_BC_SCALE_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum value of
        scale in the bc utility. Its value is 99.

``_POSIX2_BC_STRING_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum size of a
        string constant in the bc utility. Its value is 1000.

``_POSIX2_COLL_WEIGHTS_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum number of
        weights that can necessarily be used in defining the collating sequence for a
        locale. Its value is 2.

``_POSIX2_EXPR_NEST_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum number of
        expressions nested within parenthesis when using the expr utility. Its value is
        32.

``_POSIX2_LINE_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum size of a
        text line that the text utilities can handle. Its value is 2048.

``_POSIX2_EQUIV_CLASS_MAX``
        The most restrictive limit permitted by POSIX.2 for the maximum number of
        weights that can be assigned to an entry of the LC_COLLATE category ‘order’
        keyword in a locale definition. Its value is 2. The GNU C Library does not
        presently support locale definitions.

String-Valued Parameters
------------------------

**POSIX.2** defines a way to get string-valued parameters from the operating system with the
function confstr:

``size_t confstr (int parameter, char buf, size t len) [Function]``

Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety
Concepts], page 2.

This function reads the value of a string-valued system parameter, storing the string
into len bytes of memory space starting at buf. The parameter argument should be
one of the **‘_CS_’** symbols listed below.
The normal return value from confstr is the length of the string value that you asked
for. If you supply a null pointer for buf, then confstr does not try to store the string;
it just returns its length. A value of 0 indicates an error.
If the string you asked for is too long for the buffer (that is, longer than len - 1),
then confstr stores just that much (leaving room for the terminating null character).
You can tell that this has happened because confstr returns a value greater than or
equal to len.
The following errno error conditions are defined for this function:
**EINVAL** The value of the parameter is invalid.

Currently there is just one parameter you can read with confstr:

``_CS_PATH``
        This parameter’s value is the recommended default path for searching for executable
        files. This is the path that a user has by default just after logging
        in.

``_CS_LFS_CFLAGS``
        The returned string specifies which additional flags must be given to the C
        compiler if a source is compiled using the _LARGEFILE_SOURCE feature select
        macro; see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS_LDFLAGS``
        The returned string specifies which additional flags must be given to the linker
        if a source is compiled using the _LARGEFILE_SOURCE feature select macro; see
        Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS_LIBS``
        The returned string specifies which additional libraries must be linked to the
        application if a source is compiled using the _LARGEFILE_SOURCE feature select
        macro; see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS_LINTFLAGS``
        The returned string specifies which additional flags must be given to the lint
        tool if a source is compiled using the _LARGEFILE_SOURCE feature select macro;
        see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS64_CFLAGS``
        The returned string specifies which additional flags must be given to the C
        compiler if a source is compiled using the _LARGEFILE64_SOURCE feature select
        macro; see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS64_LDFLAGS``
        The returned string specifies which additional flags must be given to the linker
        if a source is compiled using the _LARGEFILE64_SOURCE feature select macro;
        see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS64_LIBS``
        The returned string specifies which additional libraries must be linked to the
        application if a source is compiled using the _LARGEFILE64_SOURCE feature
        select macro; see Section 1.3.4 [Feature Test Macros], page 15.

``_CS_LFS64_LINTFLAGS``
        The returned string specifies which additional flags must be given to the lint tool
        if a source is compiled using the _LARGEFILE64_SOURCE feature select macro;
        see Section 1.3.4 [Feature Test Macros], page 15.

The way to use confstr without any arbitrary limit on string size is to call it twice:
first call it to get the length, allocate the buffer accordingly, and then call confstr again
to fill the buffer, like this:

.. code-block:: c

   char *
   get_default_path (void)
   {
     size_t len = confstr (_CS_PATH, NULL, 0);
     char *buffer = (char *) xmalloc (len);
     if (conf str (_CS_PATH, buf, len + 1) == 0)
     {
        free (buffer);
        return NULL;
    }
    return buffer;
  }
