Inter-Process Communication
###########################

This chapter describes the GNU C Library inter-process communication primitives

==========
Semaphores
==========
The GNU C Library implements the semaphore APIs as defined in POSIX and System V.
Semaphores can be used by multiple processes to coordinate shared resources. 
The following is a complete list of the semaphore functions provided by the GNU C Library.

System V Semaphores
-------------------
``int semctl(intsemid, intsemnum, intcmd);``
    Preliminary:|MT-Safe|AS-Safe|AC-Unsafe corrupt/linux|SeeSection 1.2.2.1 POSIX Safety Concepts, page 2.

``int semget(keytkey, intnsems, intsemflg);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|SeeSection 1.2.2.1 POSIX Safety Concepts, page 2.

``int semop(intsemid, struct sembuf *sops, sizetnsops);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|SeeSection 1.2.2.1 POSIX Safety Concepts, page 2.

``int semtimedop(intsemid, struct sembuf *sops, sizetnsops, conststruct timespec *timeout);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|SeeSection 1.2.2.1 POSIX Safety Concepts, page 2.

POSIX Semaphores
----------------

``int sem_init(semt *sem, intpshared, unsigned intvalue);``
    Preliminary:|MT-Safe|AS-Safe|AC-Unsafe corrupt|SeeSection 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_destroy(semt *sem);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``sem_t *sem_open(const char *name, intoflag, ...);``
    Preliminary:|MT-Safe|AS-Unsafe  init|AC-Unsafe  init|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_close(semt *sem);``
    Preliminary:|MT-Safe|AS-Unsafe  lock|AC-Unsafe  lock|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_unlink(const char *name);``
    Preliminary:|MT-Safe|AS-Unsafe init|AC-Unsafe corrupt|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_wait(semt *sem);``
    Preliminary:|MT-Safe|AS-Safe|AC-Unsafe corrupt|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_timedwait(semt *sem, const struct timespec *abstime);``
    Preliminary:|MT-Safe|AS-Safe|AC-Unsafe corrupt|See Section 1.2.2.1 POSIX Safety Concepts, page 2.

``int sem_trywait(semt *sem);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|See Section 1.2.2.1 POSIX SafetyConcepts, page 2.

``int sem_post(semt *sem);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|See Section 1.2.2.1 POSIX SafetyConcepts, page 2.

``int sem_getvalue(semt *sem, int *sval);``
    Preliminary:|MT-Safe|AS-Safe|AC-Safe|See Section 1.2.2.1 POSIX SafetyConcepts, page 2.

