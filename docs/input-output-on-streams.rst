==========================
12 Input/Output on Streams
==========================

This chapter describes the functions for creating streams and performing input and output
operations on them. As discussed in Chapter 11 [Input/Output Overview], page 245, a stream
is a fairly abstract, high-level concept representing a communications channel to a
file, device, or process.


12.1 Streams
------------

For historical reasons, the type of the C data structure that represents a stream is called
**FILE** rather than “stream”. Since most of the library functions deal with objects of type
**FILE ***, sometimes the term file pointer is also used to mean “stream”. This leads to
unfortunate confusion over terminology in many books on C. This manual, however, is
careful to use the terms “file” and “stream” only in the technical sense.

    The **FILE** type is declared in the header file **stdio.h**

**FILE**                                                                     [Data Type]

    This is the data type used to represent stream objects. A FILE object holds all of the
    internal state information about the connection to the associated file, including such
    things as the file position indicator and buffering information. Each stream also has
    error and end-of-file status indicators that can be tested with the ferror and feof
    functions; see Section 12.15 [End-Of-File and Errors], page 304.

    **FILE** objects are allocated and managed internally by the input/output library functions.
    Don’t try to create your own objects of type FILE; let the library do it. Your programs
    should deal only with pointers to these objects (that is, FILE * values) rather than the
    objects themselves.

12.2 Standard Streams
---------------------

When the **main** function of your program is invoked, it already has three predefined streams
open and available for use. These represent the “standard” input and output channels that
have been established for the process.

    These streams are declared in the header file **stdio.h**.

**FILE * stdin**                                                               [Variable]

    The standard input stream, which is the normal source of input for the program.

**FILE * stdout**                                                              [Variable]

    The standard output stream, which is used for normal output from the program.

**FILE * stderr**                                                              [Variable]

    The standard error stream, which is used for error messages and diagnostics issued
    by the program.


  On GNU systems, you can specify what files or processes correspond to these streams
using the pipe and redirection facilities provided by the shell. (The primitives shells use to
implement these facilities are described in Chapter 14 [File System Interface], page 379.)


*Chapter 12: Input/Output on Streams*                                                    *251*

Most other operating systems provide similar mechanisms, but the details of how to use
them can vary.

  In the GNU C Library, stdin, stdout, and stderr are normal variables which you can set
just like any others. For example, to redirect the standard output to a file, you could
do:

    **fclose (stdout);
    stdout = fopen ("standard-output-file", "w");**

  Note however, that in other systems stdin, stdout, and stderr are macros that you cannot
assign to in the normal way. But you can use freopen to get the effect of closing one and
reopening it. See Section 12.3 [Opening Streams], page 251.

  The three streams stdin, stdout, and stderr are not unoriented at program start (see
Section 12.6 [Streams in Internationalized Applications], page 259).


12.3 Opening Streams
--------------------

Opening a file with the fopen function creates a new stream and establishes a connection
between the stream and a file. This may involve creating a new file.

    Everything described in this section is declared in the header file **stdio.h**.

**FILE** * fopen (const char *\*filename*, const char *opentype)                [Function]

   Preliminary: | MT-Safe | AS-Unsafe heap lock | AC-Unsafe mem fd lock | See Section 1.2.2.1
   [POSIX Safety Concepts], page 2.

   The **fopen** function opens a stream for I/O to the file filename, and returns a pointer
   to the stream.

   The opentype argument is a string that controls how the file is opened and specifies
   attributes of the resulting stream. It must begin with one of the following sequences
   of characters:


   *r*		 Open an existing file for reading only.

   *w* 		 Open the file for writing only. If the file already exists, it is truncated to
		 zero length. Otherwise a new file is created.

   *a* 		 Open a file for append access; that is, writing at the end of file only. If
 		 the file already exists, its initial contents are unchanged and output to
 		 the stream is appended to the end of the file. Otherwise, a new, empty
                 file is created.

   *r+* 	 Open an existing file for both reading and writing. The initial contents
                 of the file are unchanged and the initial file position is at the beginning
		 of the file.

   *w+*		 Open a file for both reading and writing. If the file already exists, it is
                 truncated to zero length. Otherwise, a new file is created.

   *a+*		 Open or create file for both reading and appending. If the file exists,
		 its initial contents are unchanged. Otherwise, a new file is created. The
		 initial file position for reading is at the beginning of the file, but output
	         is always appended to the end of the file.


*Chapter 12: Input/Output on Streams* 								*252*


As you can see, *+* requests a stream that can do both input and output. When using
such a stream, you must call fflush (see Section 12.20 [Stream Buffering], page 312)
or a file positioning function such as fseek (see Section 12.18 [File Positioning],
page 307) when switching from reading to writing or vice versa. Otherwise, internal
buffers might not be emptied properly.

Additional characters may appear after these to specify flags for the call. Always
put the mode (*r*, *w+*, etc.) first; that is the only part you are guaranteed will be
understood by all systems.

The GNU C Library defines additional characters for use in opentype:

   *c*   	The file is opened with cancellation in the I/O functions disabled.

   *e* 		The underlying file descriptor will be closed if you use any of the exec...
		functions (see Section 26.5 [Executing a File], page 755). (This is equivalent
		to having set FD_CLOEXEC on that descriptor. See Section 13.13 [File
		Descriptor Flags], page 363.)

   *m* 		The file is opened and accessed using mmap. This is only supported with
		files opened for reading.

   *x* 		Insist on creating a new file—if a file filename already exists, fopen fails
		rather than opening it. If you use *x* you are guaranteed that you will not
		clobber an existing file. This is equivalent to the O_EXCL option to the
		open function (see Section 13.1 [Opening and Closing Files], page 325).

		The *x* modifier is part of ISO C11.

The character *b* in opentype has a standard meaning; it requests a binary stream
rather than a text stream. But this makes no difference in POSIX systems (including
GNU systems). If both *+* and *b* are specified, they can appear in either order. See
Section 12.17 [Text and Binary Streams], page 306.


If the *opentype* string contains the sequence ,ccs=*STRING* then **STRING** is taken as
the name of a coded character set and fopen will mark the stream as wide-oriented
with appropriate conversion functions in place to convert from and to the character
set **STRING**. Any other stream is opened initially unoriented and the orientation is
decided with the first file operation. If the first operation is a wide character operation,
the stream is not only marked as wide-oriented, also the conversion functions to
convert to the coded character set used for the current locale are loaded. This will
not change anymore from this point on even if the locale selected for the **LC_CTYPE**
category is changed.

  You can have multiple streams (or file descriptors) pointing to the same file open at the
same time. If you do only input, this works straightforwardly, but you must be careful if any


*Chapter 12: Input/Output on Streams*                                               *253*


output streams are included. See Section 13.5 [Dangers of Mixing Streams and Descriptors],
page 336. This is equally true whether the streams are in one program (not usual) or in
several programs (which can easily happen). It may be advantageous to use the file locking
facilities to avoid simultaneous access. See Section 13.15 [File Locks], page 370.


**FILE** * **fopen64**(const char * **filename**, const char * **opentype**) 			   [Function]

     Preliminary: | MT-Safe | AS-Unsafe heap lock | AC-Unsafe mem fd lock | See
     Section 1.2.2.1 [POSIX Safety Concepts], page 2.


    This function is similar to fopen but the stream it returns a pointer for is opened
    using open64. Therefore this stream can be used even on files larger than 231 bytes
    on 32 bit machines.

    Please note that the return type is still FILE *. There is no special FILE type for the
    LFS interface.

    If the sources are compiled with _FILE_OFFSET_BITS == 64 on a 32 bits machine this
    function is available under the name fopen and so transparently replaces the old
    interface.


int **FOPEN_MAX** 									   [Macro]

    The value of this macro is an integer constant expression that represents the minimum
    number of streams that the implementation guarantees can be open simultaneously.
    You might be able to open more than this many streams, but that is not guaranteed.
    The value of this constant is at least eight, which includes the three standard streams
    **stdin, stdout,** and **stderr**. In **POSIX.1** systems this value is determined by the
    **OPEN_MAX** parameter; see Section 32.1 [General Capacity Limits], page 841. In **BSD**
    and **GNU**, it is controlled by the **RLIMIT_NOFILE** resource limit; see Section 22.2
    [Limiting Resource Usage], page 635.

**FILE** * **freopen** (const char *\**filename**, const char *\**opentype**,**FILE** *\**stream**)         [Funcion]

    Preliminary: | MT-Safe | AS-Unsafe corrupt | AC-Unsafe corrupt fd | See Section 1.2.2.1 [POSIX Safety
    Concepts], page 2.

    This function is like a combination of **fclose** and **fopen**. It first closes the stream
    referred to by stream, ignoring any errors that are detected in the process. (Because
    errors are ignored, you should not use **freopen** on an output stream if you have
    actually done any output using the stream.) Then the file named by *filename* is
    opened with mode opentype as for **fopen**, and associated with the same stream object
    stream.

    If the operation fails, a null pointer is returned; otherwise, **freopen** returns stream.

    If the operation fails, a null pointer is returned; otherwise, **freopen** returns stream.
    freopen has traditionally been used to connect a standard stream such as stdin with
    a file of your own choice. This is useful in programs in which use of a standard stream
    for certain purposes is hard-coded. In the **GNU C** Library, you can simply close the
    standard streams and open new ones with fopen. But other systems lack this ability,
    so using freopen is more portable.

    When the sources are compiling with **_FILE_OFFSET_BITS == 64** on a 32 bit machine
    this function is in fact **freopen64** since the LFS interface replaces transparently the
    old interface.


*Chapter 12: Input/Output on Streams* 								*254*


**FILE** * **freopen64** (const char * **filename**, const char * **opentype**,**FILE** * **stream**)           [Function]

    Preliminary: | MT-Safe | AS-Unsafe corrupt | AC-Unsafe corrupt fd | See Section 1.2.2.1
    [POSIX Safety Concepts], page 2.

    This function is similar to freopen. The only difference is that on 32 bit machine the
    stream returned is able to read beyond the 2**31 bytes limits imposed by the normal
    interface. It should be noted that the stream pointed to by stream need not be opened
    using **fopen64** or **freopen64** since its mode is not important for this function.

    If the sources are compiled with **_FILE_OFFSET_BITS == 64** on a 32 bits machine this
    function is available under the name **freopen** and so transparently replaces the old
    interface.

  In some situations it is useful to know whether a given stream is available for reading
or writing. This information is normally not available and would have to be remembered
separately. Solaris introduced a few functions to get this information from the stream
descriptor and these functions are also available in the **GNU C** Library.


**int __freadable** (**FILE** * **stream**) 									[Function]

    Preliminary: | MT-Safe | AS-Safe | AC-Safe | See Section 1.2.2.1 [POSIX Safety
    Concepts], page 2.

    The **__freadable** function determines whether the stream stream was opened to
    allow reading. In this case the return value is nonzero. For write-only streams the
    function returns zero.

    This function is declared in **stdio_ext.h**.


