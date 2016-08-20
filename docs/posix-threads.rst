=============
POSIX Threads
=============

This chapter describes the GNU C Library POSIX Thread implementation.

Thread-specific Data
====================

The GNU C Library implements functions to allow users to create and manage data specific
to a thread. Such data may be destroyed at thread exit, if a destructor is provided. The
following functions are defined:


int pthread_key_create ( pthread key t *key, void
        ( *destructor )( void* ))
    Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety
    Concepts], page 2.
    Create a thread-specific data key for the calling thread, referenced by key.
    Objects declared with the C++11 thread_local keyword are destroyed before thread-
    specific data, so they should not be used in thread-specific data destructors or even
    as members of the thread-specific data, since the latter is passed as an argument to
    the destructor function.

int pthread_key_delete ( pthread key t key )
    Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety
    Concepts], page 2.

    Destroy the thread-specific data key in the calling thread. The destructor for the
    thread-specific data is not called during destruction, nor is it called during thread
    exit.

void *pthread_getspecific ( pthread key t key )
    Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety
    Concepts], page 2.

    Return the thread-specific data associated with key in the calling thread.

int pthread_setspecific ( pthread key t key, const void *value )
    Preliminary: | MT-Safe | AS-Unsafe corrupt heap | AC-Unsafe corrupt mem | See
    Section 1.2.2.1 [POSIX Safety Concepts], page 2.

   Associate the thread-specific value with key in the calling thread.


====================
Non-POSIX Extensions
====================

In addition to implementing the POSIX API for threads, the GNU C Library provides
additional functions and interfaces to provide functionality not specified in the standard.

Setting Process-wide defaults for thread attributes
===================================================

The GNU C Library provides non-standard API functions to set and get the default at-
tributes used in the creation of threads in a process.

int pthread_getattr_default_np ( pthread attr t *attr )
    Preliminary: | MT-Safe | AS-Unsafe lock | AC-Unsafe lock | See Section 1.2.2.1
    [POSIX Safety Concepts], page 2.

    Get the default attribute values and set attr to match. This function returns 0 on
    success and a non-zero error code on failure.

int pthread_setattr_default_np ( pthread attr t *attr )
    Preliminary: | MT-Safe | AS-Unsafe heap lock | AC-Unsafe lock mem | See
    Section 1.2.2.1 [POSIX Safety Concepts], page 2.

    Set the default attribute values to match the values in attr. The function returns 0
    on success and a non-zero error code on failure. The following error codes are defined
    for this function:

    EINVAL    At least one of the values in attr does not qualify as valid for the attributes
    or the stack address is set in the attribute.

    ENOMEM    The system does not have sufficient memory.
