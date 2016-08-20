===============
Internal probes
===============

In order to aid in debugging and monitoring internal behavior, the GNU C Library exposes
nearly-zero-overhead SystemTap probes marked with the libc provider.
These probes are not part of the GNU C Library stable ABI, and they are subject to
change or removal across releases. Our only promise with regard to them is that, if we
find a need to remove or modify the arguments of a probe, the modified probe will have a
different name, so that program monitors relying on the old probe will not get unexpected
arguments.

Memory Allocation Probes
------------------------

These probes are designed to signal relatively unusual situations within the virtual memory
subsystem of the GNU C Library.

memory_sbrk_more (void *$arg1, size t $arg2) [Probe]

This probe is triggered after the main arena is extended by calling sbrk. Argument
$arg1 is the additional size requested to sbrk, and $arg2 is the pointer that marks
the end of the sbrk area, returned in response to the request.

memory_sbrk_less (void *$arg1, size t $arg2) [Probe]

This probe is triggered after the size of the main arena is decreased by calling sbrk.
Argument $arg1 is the size released by sbrk (the positive value, rather than the
negative value passed to sbrk), and $arg2 is the pointer that marks the end of the
sbrk area, returned in response to the request.

memory_heap_new (void *$arg1, size t $arg2) [Probe]

This probe is triggered after a new heap is mmaped. Argument $arg1 is a pointer to
the base of the memory area, where the heap_info data structure is held, and $arg2
is the size of the heap.

memory_heap_free (void *$arg1, size t $arg2) [Probe]

This probe is triggered before (unlike the other sbrk and heap probes) a heap is
completely removed via munmap. Argument $arg1 is a pointer to the heap, and $arg2
is the size of the heap.

memory_heap_more (void *$arg1, size t $arg2) [Probe]

This probe is triggered after a trailing portion of an mmaped heap is extended. Argument
$arg1 is a pointer to the heap, and $arg2 is the new size of the heap.

memory_heap_less (void *$arg1, size t $arg2) [Probe]

This probe is triggered after a trailing portion of an mmaped heap is released. Argument
$arg1 is a pointer to the heap, and $arg2 is the new size of the heap.

memory_malloc_retry (size t $arg1) [Probe]

memory_realloc_retry (size t $arg1, void *$arg2) [Probe]

memory_memalign_retry (size t $arg1, size t $arg2) [Probe]

memory_calloc_retry (size t $arg1) [Probe]

These probes are triggered when the corresponding functions fail to obtain the requested
amount of memory from the arena in use, before they call arena_get_retry 
to select an alternate arena in which to retry the allocation. Argument $arg1 is the
amount of memory requested by the user; in the calloc case, that is the total size
computed from both function arguments. In the realloc case, $arg2 is the pointer
to the memory area being resized. In the memalign case, $arg2 is the alignment to
be used for the request, which may be stricter than the value passed to the memalign
function. A memalign probe is also used by functions posix_memalign, valloc and
pvalloc.

Note that the argument order does not match that of the corresponding two-argument
functions, so that in all of these probes the user-requested allocation size is in $arg1.

memory_arena_retry (size t $arg1, void *$arg2) [Probe]

This probe is triggered within arena_get_retry (the function called to select the
alternate arena in which to retry an allocation that failed on the first attempt), before
the selection of an alternate arena. This probe is redundant, but much easier to use
when it’s not important to determine which of the various memory allocation functions
is failing to allocate on the first try. Argument $arg1 is the same as in the functionspecific
probes, except for extra room for padding introduced by functions that have
to ensure stricter alignment. Argument $arg2 is the arena in which allocation failed.

memory_arena_new (void *$arg1, size t $arg2) [Probe]

This probe is triggered when malloc allocates and initializes an additional arena (not
the main arena), but before the arena is assigned to the running thread or inserted into
the internal linked list of arenas. The arena’s malloc_state internal data structure
is located at $arg1, within a newly-allocated heap big enough to hold at least $arg2
bytes.

memory_arena_reuse (void *$arg1, void *$arg2) [Probe]

This probe is triggered when malloc has just selected an existing arena to reuse,
and (temporarily) reserved it for exclusive use. Argument $arg1 is a pointer to the
newly-selected arena, and $arg2 is a pointer to the arena previously used by that
thread.
This occurs within reused_arena, right after the mutex mentioned in probe memory_
arena_reuse_wait is acquired; argument $arg1 will point to the same arena. In this
configuration, this will usually only occur once per thread. The exception is when a
thread first selected the main arena, but a subsequent allocation from it fails: then,
and only then, may we switch to another arena to retry that allocations, and for
further allocations within that thread.

memory_arena_reuse_wait (void *$arg1, void *$arg2, void *$arg3) [Probe]

This probe is triggered when malloc is about to wait for an arena to become available
for reuse. Argument $arg1 holds a pointer to the mutex the thread is going to wait
on, $arg2 is a pointer to a newly-chosen arena to be reused, and $arg3 is a pointer
to the arena previously used by that thread.
This occurs within reused_arena, when a thread first tries to allocate memory or needs a retry after a failure to allocate from the main arena, there isn’t any free arena, the maximum number of arenas has been reached, and an existing arena was chosen for reuse, but its mutex could not be immediately acquired. The mutex in $arg1 is the mutex of the selected arena.

memory_arena_reuse_free_list (void *$arg1) [Probe]

This probe is triggered when malloc has chosen an arena that is in the free list for
use by a thread, within the get_free_list function. The argument $arg1 holds a
pointer to the selected arena.

memory_mallopt (int $arg1, int $arg2) [Probe]

This probe is triggered when function mallopt is called to change malloc internal
configuration parameters, before any change to the parameters is made. The arguments
$arg1 and $arg2 are the ones passed to the mallopt function.

memory_mallopt_mxfast (int $arg1, int $arg2) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_MXFAST, and the requested value is in an acceptable range. Argument
$arg1 is the requested value, and $arg2 is the previous value of this malloc
parameter.

memory_mallopt_trim_threshold (int $arg1, int $arg2, int $arg3) [Probe]

This probe is triggere shortly after the memory_mallopt probe, when the parameter
to be changed is M_TRIM_THRESHOLD. Argument $arg1 is the requested value, $arg2
is the previous value of this malloc parameter, and $arg3 is nonzero if dynamic
threshold adjustment was already disabled.

memory_mallopt_top_pad (int $arg1, int $arg2, int $arg3) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_TOP_PAD. Argument $arg1 is the requested value, $arg2 is the
previous value of this malloc parameter, and $arg3 is nonzero if dynamic threshold
adjustment was already disabled.

memory_mallopt_mmap_threshold (int $arg1, int $arg2, int $arg3) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_MMAP_THRESHOLD, and the requested value is in an acceptable
range. Argument $arg1 is the requested value, $arg2 is the previous value of this
malloc parameter, and $arg3 is nonzero if dynamic threshold adjustment was already
disabled.

memory_mallopt_mmap_max (int $arg1, int $arg2, int $arg3) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_MMAP_MAX. Argument $arg1 is the requested value, $arg2 is the
previous value of this malloc parameter, and $arg3 is nonzero if dynamic threshold
adjustment was already disabled.

memory_mallopt_check_action (int $arg1, int $arg2) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_CHECK_ACTION. Argument $arg1 is the requested value, and $arg2
is the previous value of this malloc parameter.

memory_mallopt_perturb (int $arg1, int $arg2) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_PERTURB. Argument $arg1 is the requested value, and $arg2 is
the previous value of this malloc parameter.

memory_mallopt_arena_test (int $arg1, int $arg2) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_ARENA_TEST, and the requested value is in an acceptable range.
Argument $arg1 is the requested value, and $arg2 is the previous value of this malloc
parameter.

memory_mallopt_arena_max (int $arg1, int $arg2) [Probe]

This probe is triggered shortly after the memory_mallopt probe, when the parameter
to be changed is M_ARENA_MAX, and the requested value is in an acceptable range.
Argument $arg1 is the requested value, and $arg2 is the previous value of this malloc
parameter.

memory_mallopt_free_dyn_thresholds (int $arg1, int $arg2) [Probe]

This probe is triggered when function free decides to adjust the dynamic brk/mmap
thresholds. Argument $arg1 and $arg2 are the adjusted mmap and trim thresholds,
respectively.

Mathematical Function Probes
----------------------------

Some mathematical functions fall back to multiple precision arithmetic for some inputs to
get last bit precision for their return values. This multiple precision fallback is much slower
than the default algorithms and may have a significant impact on application performance.
The systemtap probe markers described in this section may help you determine if your
application calls mathematical functions with inputs that may result in multiple-precision
arithmetic.

Unless explicitly mentioned otherwise, a precision of 1 implies 24 bits of precision in the
mantissa of the multiple precision number. Hence, a precision level of 32 implies 768 bits
of precision in the mantissa.

slowexp_p6 (double $arg1, double $arg2) [Probe]

This probe is triggered when the exp function is called with an input that results in
multiple precision computation with precision 6. Argument $arg1 is the input value
and $arg2 is the computed output.

slowexp_p32 (double $arg1, double $arg2) [Probe]

This probe is triggered when the exp function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input value
and $arg2 is the computed output.

slowpow_p10 (double $arg1, double $arg2, double $arg3, double [Probe]
$arg4)

This probe is triggered when the pow function is called with inputs that result in
multiple precision computation with precision 10. Arguments $arg1 and $arg2 are
the input values, $arg3 is the value computed in the fast phase of the algorithm and
$arg4 is the final accurate value.

slowpow_p32 (double $arg1, double $arg2, double $arg3, double [Probe]
$arg4)

This probe is triggered when the pow function is called with an input that results in
multiple precision computation with precision 32. Arguments $arg1 and $arg2 are
the input values, $arg3 is the value computed in the fast phase of the algorithm and
$arg4 is the final accurate value.

slowlog (int $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the log function is called with an input that results
in multiple precision computation. Argument $arg1 is the precision with which the
computation succeeded. Argument $arg2 is the input and $arg3 is the computed
output.

slowlog_inexact (int $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the log function is called with an input that results in
multiple precision computation and none of the multiple precision computations result
in an accurate result. Argument $arg1 is the maximum precision with which computations
were performed. Argument $arg2 is the input and $arg3 is the computed
output.

slowatan2 (int $arg1, double $arg2, double $arg3, double $arg4) [Probe]

This probe is triggered when the atan2 function is called with an input that results
in multiple precision computation. Argument $arg1 is the precision with which computation
succeeded. Arguments $arg2 and $arg3 are inputs to the atan2 function
and $arg4 is the computed result.

slowatan2_inexact (int $arg1, double $arg2, double $arg3, double [Probe]
$arg4)

This probe is triggered when the atan function is called with an input that results
in multiple precision computation and none of the multiple precision computations
result in an accurate result. Argument $arg1 is the maximum precision with which
computations were performed. Arguments $arg2 and $arg3 are inputs to the atan2
function and $arg4 is the computed result.

slowatan (int $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the atan function is called with an input that results in
multiple precision computation. Argument $arg1 is the precision with which computation
succeeded. Argument $arg2 is the input to the atan function and $arg3 is the
computed result.

slowatan_inexact (int $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the atan function is called with an input that results
in multiple precision computation and none of the multiple precision computations
result in an accurate result. Argument $arg1 is the maximum precision with which
computations were performed. Argument $arg2 is the input to the atan function and
$arg3 is the computed result.

slowtan (double $arg1, double $arg2) [Probe]

This probe is triggered when the tan function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input to
the function and $arg2 is the computed result.

slowasin (double $arg1, double $arg2) [Probe]

This probe is triggered when the asin function is called with an input that results
in multiple precision computation with precision 32. Argument $arg1 is the input to
the function and $arg2 is the computed result.

slowacos (double $arg1, double $arg2) [Probe]

This probe is triggered when the acos function is called with an input that results
in multiple precision computation with precision 32. Argument $arg1 is the input to
the function and $arg2 is the computed result.

slowsin (double $arg1, double $arg2) [Probe]

This probe is triggered when the sin function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input to
the function and $arg2 is the computed result.

slowcos (double $arg1, double $arg2) [Probe]

This probe is triggered when the cos function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input to
the function and $arg2 is the computed result.

slowsin_dx (double $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the sin function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input to
the function, $arg2 is the error bound of $arg1 and $arg3 is the computed result.

slowcos_dx (double $arg1, double $arg2, double $arg3) [Probe]

This probe is triggered when the cos function is called with an input that results in
multiple precision computation with precision 32. Argument $arg1 is the input to
the function, $arg2 is the error bound of $arg1 and $arg3 is the computed result.

Non-local Goto Probes
---------------------

These probes are used to signal calls to setjmp, sigsetjmp, longjmp or siglongjmp.

setjmp (void *$arg1, int $arg2, void *$arg3) [Probe]

This probe is triggered whenever setjmp or sigsetjmp is called. Argument $arg1 is
a pointer to the jmp_buf passed as the first argument of setjmp or sigsetjmp, $arg2
is the second argument of sigsetjmp or zero if this is a call to setjmp and $arg3 is
a pointer to the return address that will be stored in the jmp_buf.

longjmp (void *$arg1, int $arg2, void *$arg3) [Probe]

This probe is triggered whenever longjmp or siglongjmp is called. Argument $arg1
is a pointer to the jmp_buf passed as the first argument of longjmp or siglongjmp,
$arg2 is the return value passed as the second argument of longjmp or siglongjmp
and $arg3 is a pointer to the return address longjmp or siglongjmp will return to.
The longjmp probe is triggered at a point where the registers have not yet been
restored to the values in the jmp_buf and unwinding will show a call stack including
the caller of longjmp or siglongjmp.

longjmp_target (void *$arg1, int $arg2, void *$arg3) [Probe]

This probe is triggered under the same conditions and with the same arguments as
the longjmp probe.
The longjmp_target probe is triggered at a point where the registers have been
restored to the values in the jmp_buf and unwinding will show a call stack including
the caller of setjmp or sigsetjmp.
