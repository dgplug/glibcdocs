=================
Debugging support
=================

Applications are usually debugged using dedicated debugger programs. But sometimes
this is not possible and, in any case, it is useful to provide the developer with as much
information as possible at the time the problems are experienced. For this reason a few
functions are provided which a program can use to help the developer more easily locate
the problem.

Backtraces
==========

A *backtrace* is a list of the function calls that are currently active in a thread. The usual way
to inspect a backtrace of a program is to use an external debugger such as gdb. However,
sometimes it is useful to obtain a backtrace programmatically from within a program, e.g.,
for the purposes of logging or diagnostics.

	The header file *execinfo.h* declares three functions that obtain and manipulate back-
	traces of the current thread.

*int backtrace ( void **buffer, int size )				[Function]*
	Preliminary: | MT-Safe | AS-Unsafe init heap dlopen plugin lock | AC-Unsafe init
	mem lock fd | See Section 1.2.2.1 [POSIX Safety Concepts], page 2.

	The *backtrace* function obtains a backtrace for the current thread, as a list of point-
	ers, and places the information into buffer. The argument size should be the number
	of void * elements that will fit into buffer. The return value is the actual number of
	entries of *buffer* that are obtained, and is at most size.

	The pointers placed in *buffer* are actually return addresses obtained by inspecting
	the stack, one return address per stack frame.

	Note that certain compiler optimizations may interfere with obtaining a valid back-
	trace. Function inlining causes the inlined function to not have a stack frame; tail
	call optimization replaces one stack frame with another; frame pointer elimination
	will stop *backtrace* from interpreting the stack contents correctly.

*char ** backtrace_symbols ( void *const *buffer, int size )		[Function]*
	Preliminary: | MT-Safe | AS-Unsafe heap | AC-Unsafe mem lock | See
	Section 1.2.2.1 [POSIX Safety Concepts], page 2.

	The *backtrace_symbols* function translates the information obtained from the
	backtrace function into an array of strings. The argument buffer should be a
	pointer to an array of addresses obtained via the *backtrace* function, and size is the
	number of entries in that array (the return value of *backtrace*).

	The return value is a pointer to an array of strings, which has size entries just like
	the array *buffer*. Each string contains a printable representation of the corresponding
	element of *buffer*. It includes the function name (if this can be determined), an offset
	into the function, and the actual return address (in hexadecimal).

	Currently, the function name and offset only be obtained on systems that use the ELF
	binary format for programs and libraries. On other systems, only the hexadecimal
	return address will be present. Also, you may need to pass additional flags to the
	linker to make the function names available to the program. (For example, on systems
	using GNU ld, you must pass (**-rdynamic**.)

	The return value of *backtrace_symbols* is a pointer obtained via the *malloc* function,
	and it is the responsibility of the caller to *free* that pointer. Note that only the return
	value need be freed, not the individual strings.
	The return value is *NULL* if sufficient memory for the strings cannot be obtained.

*void backtrace_symbols_fd ( void *const *buffer, int size, int fd )	[Function]*
	Preliminary: | MT-Safe | AS-Safe | AC-Unsafe lock | See Section 1.2.2.1 [POSIX
	Safety Concepts], page 2.

	The *backtrace_symbols_fd* function performs the same translation as the function
	*backtrace_symbols* function. Instead of returning the strings to the caller, it writes
	the strings to the file descriptor fd, one per line. It does not use the malloc function,
	and can therefore be used in situations where that function might fail.


The following program illustrates the use of these functions. Note that the array to
contain the return addresses returned by *backtrace* is allocated on the stack. Therefore
code like this can be used in situations where the memory handling via malloc does not
work anymore (in which case the *backtrace_symbols* has to be replaced by a *backtrace_
symbols_fd* call as well). The number of return addresses is normally not very large. Even
complicated programs rather seldom have a nesting level of more than, say, 50 and with
200 possible entries probably all programs should be covered.


.. code-block:: c

	#include <execinfo.h>
	#include <stdio.h>
	#include <stdlib.h>

	/* Obtain a backtrace and print it to stdout. */
	void
	print_trace (void)
	{
	 void *array[10];
	 size_t size;
	 char **strings;
	 size_t i;

	 size = backtrace (array, 10);
	 strings = backtrace_symbols (array, size);

	 printf ("Obtained %zd stack frames.\n", size);

	 for (i = 0; i < size; i++)
		 printf ("%s\n", strings[i]);
	
	 free (strings);
	}

	/* A dummy function to make the backtrace more interesting. */
	void
	dummy_function (void)
	{
	 print_trace ();
	}

	int
	main (void)
	{
	 dummy_function ();
         return 0;
	}
