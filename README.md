Introduction
============

Consistent hashing is a scheme that provides hash table functionality in a way that the addition or removal of one slot does not significantly change the mapping of keys to slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped. By using consistent hashing, only K/n keys need to be remapped on average, where K is the number of keys, and n is the number of slots.

CHash is Dailymotion's implementation of consistent hashing, available as a light and fast C library (both static and shared objects provided), with PHP and Python bindings.

Useful pointers
===============

Below are some useful pointers pertaining to consistent hashing in general and the CHash implementation in particular:

* The [Wikipedia definition page](http://en.wikipedia.org/wiki/Consistent_hashing)
* An  [interesting blog entry](http://www.spiteful.com/2008/03/17/programmers-toolbox-part-3-consistent-hashing) on consistent hashing
* Another [blog entry](http://weblogs.java.net/blog/tomwhite/archive/2007/11/consistent_hash.html) on the same subject
* The [MurmurHash hashing algorithm](http://murmurhash.googlepages.com) used in CHash
* The CHash [GIT repository](http://github.com/dailymotion/chash)

Compilation and installation
============================

C libraries
-----------

The base C libraries are generated by invoking the following commands:

    git clone git://github.com/dailymotion/chash.git
    cd chash/library
    make

The corresponding Debian packages (libchash1 and libchash-dev) are then generated by invoking the following command:

    make deb

Installing the generated packages is performed via the following commands (where <arch> is either i386 or amd64):

    cd ..
    sudo dpkg -i libchash1_1.0.0_<arch>.deb
    sudo dpkg -i libchash-dev_1.0.0_<arch>.deb

Please note that the packages will only be generated if the C libraries unitary tests pass successfully.

PHP extension
-------------

The PHP extension is generated by invoking the following commands:

    git clone git://github.com/dailymotion/chash.git
    cd chash/php
    sudo make -f Makefile.chash

The corresponding Debian package (php5-chash) is then generated by invoking the following command:

    sudo make -f Makefile.chash deb

Installing the generated package is performed via the following commands (where <arch> is either i386 or amd64):

    cd ..
    sudo dpkg -i php5-chash_1.0.0_<arch>.deb

Please note that the packages will only be generated if the PHP extension unitary tests pass successfully.

Python extension
----------------

TODO

C API
=====

General usage
-------------

The C API evolves around a unique structure (CHASH_CONTEXT) and a bunch of functions performing
action on that structure (mimicking an object-oriented behavior) . To use this API, you must first
include [libchash.h](http://git.dailymotion.com/component/chash.git/?a=blob;f=library/libchash.h),
declare an instance of the CHASH_CONTEXT structure, initialize it (using chash_initialize) then use
the various functions as needed. The final compiled code must be linked to the libchash.a or libchash.so
library (you'll probably need to pass the -lchash option to your linker).

A very basic example is provided below:

    #include <stdio.h>
    #include <libchash.h>

    int main()
    {
        CHASH_CONTEXT context;
        char          *target;

        chash_initialize(&context, 0);
        chash_add_target(&context, "target001", 1);
        chash_add_target(&context, "target002", 1);
        chash_add_target(&context, "target003", 1);
        chash_add_target(&context, "target004", 1);
        chash_add_target(&context, "target005", 1);
        chash_lookup_balance(&context, "candidate001", 3, &target);
        printf("%s\n", target);
        chash_terminate(&context, 0);
        return 0;
    }

Assuming the above example code was saved under sample.c, it can be compiled using the following command:

    gcc -Wall -O3 sample.c -o sample -lchash

Error codes
-----------

Most of the C API functions return an error code (a negative integer) when an error occurs.
The list of the possible values and their meaning is provided below:

* *CHASH_ERROR_DONE* (0)
The function executed properly.

* *CHASH_ERROR_MEMORY* (-1)
A memory allocation error occurred.

* *CHASH_ERROR_IO* (-2)
A file I/O error occurred.

* *CHASH_ERROR_INVALID_PARAMETER* (-10)
An invalid parameter was provided.

* *CHASH_ERROR_ALREADY_INITIALIZED* (-11)
The context is already initialized.

* *CHASH_ERROR_NOT_INITIALIZED* (-12)
The requested action cannot be performed because the context was not properly initialized.

* *CHASH_ERROR_NOT_FOUND* (-13)
The requested data cannot be found.

Functions list
--------------

### int chash_initialize(CHASH_CONTEXT *context, u_char *force)

#### Description
Initialize the given context for future use. This function *MUST* be called before using the context
with other library functions (a *CHASH_ERROR_NOT_INITIALIZED* error will be returned otherwise).

#### Parameters
* *context*: pointer to a pre-declared context
* *force*: reserved for internal use only, always 0

#### Return value
* *CHASH_ERROR_DONE*: context was successfully initialized|
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function|
* *CHASH_ERROR_ALREADY_INITIALIZED*: the context was previously initialized and the *force* parameter
  is 0 (use *chash_terminate()* first)|

### int chash_terminate(CHASH_CONTEXT *context, u_char force)

#### Description
Release the given context (freeing allocated memory as needed). This function *MUST* be called when the context is no longer used (memory leaks may occur otherwise).

#### Parameters
* *context*: pointer to an initialized context
* *force*: reserved for internal use only, always 0

#### Return value
* *CHASH_ERROR_DONE*: context was successfully terminated
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not previously initialized and the *force* parameter is 0 (use *chash_initialize()* first)

### int chash_add_target(CHASH_CONTEXT *context, const char *name, u_char weight)

#### Description
Add a new target to the given context. If the target already exists, its weight is updated according to the *weight* parameter.

#### Parameters
* *context*:pointer to an initialized context
* *name*: NULL-terminated target name
* *weight*: target weight (between 0 and 100)

#### Return value
* *CHASH_ERROR_DONE*: target was successfully added
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int chash_remove_target(CHASH_CONTEXT *context, const char *name)

#### Description
Remove a target from the given context.

#### Parameters
* *context*: pointer to an initialized context
* *name*: NULL-terminated target name

#### Return value
* *CHASH_ERROR_DONE*: target was successfully removed
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_NOT_FOUND*: the specified target does not exist in the given context

### int chash_clear_targets(CHASH_CONTEXT *context)

#### Description
Clear all targets from the given context.

#### Parameters
* *context*: pointer to an initialized context

#### Return value
* *CHASH_ERROR_DONE*: all targets were successfully removed
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)

### int chash_targets_count(CHASH_CONTEXT *context)

#### Description
Return the number of targets in the given context.

#### Parameters
* *context*: pointer to an initialized context

#### Return value
* *n*: when successful, number of targets
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)

### int chash_serialize(CHASH_CONTEXT *context, u_char **output)

#### Description
Serialize the given context state into an opaque buffer (the context state can be restored by passing this buffer back to *chash_unserialize()*).

#### Parameters
* *context*: pointer to an initialized context
* *output*: allocated serialized data (this data *MUST* be freed using the free() function when no longer used, memory leaks may occur otherwise)

#### Return value
* *n*: when successful, size in bytes of the serialized data
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_NOT_FOUND*: no target exist in the given context (use *chash_add_target()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int chash_unserialize(CHASH_CONTEXT *context, const u_char *input, u_int32_t size)

#### Description
Restore a previously serialized context state (using *chash_serialize()*).

#### Parameters
* *context*: pointer to an initialized context
* *input*: serialized data (generated using chash_serialize())
* *size*: serialized data size in bytes

#### Return value
* *n*: when successful, number of items in the newly restored continuum
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_FOUND*: no target exist in the serialized data (i.e. the serialized data is not coherent)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int chash_file_serialize(CHASH_CONTEXT *context, const char *path)

#### Description
Serialize the given context state into a file (the context state can be restored by passing this file path back to *chash_file_unserialize()*).

#### Parameters
* *context*: pointer to an initialized context
* *path*: path of the file to which the context state will be saved

#### Return value
* *n*: when successful, size in bytes of the serialized data
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_NOT_FOUND*: no target exist in the given context (use *chash_add_target()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred
* *CHASH_ERROR_IO*: an I/O error occurred (i.e. the given file path couldn't be written to)

### int chash_file_unserialize(CHASH_CONTEXT *context, const char *path)

#### Description
Restore a previously serialized context state (using *chash_file_serialize()*).

#### Parameters
* *context*: pointer to an initialized context
* *path*: path of the file from which the context state will be restored

#### Return value
* *n*: when successful, number of items in the newly restored continuum
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_FOUND*: no target exist in the serialized data (i.e. the serialized data is not coherent)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred
* *CHASH_ERROR_IO*: an I/O error occurred (i.e. the given file path couldn't be read from)

### int chash_lookup(CHASH_CONTEXT *context, const char *name, u_int16_t count, char ***output)

#### Description
Perform a lookup in the given context, returning *count* distincts targets.

#### Parameters
* *context*: pointer to an initialized context
* *name*: NULL-terminated candidate name
* *count*: desired targets count
* *output*: matching targets list (read-only array, *MUST* not be modified by calling code)

#### Return value
* *n*: when successful, count of returned matching targets
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_NOT_FOUND*: no target exist in the given context (use *chash_add_target()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int chash_lookup_balance(CHASH_CONTEXT *context, const char *name, u_int16_t count, char **output)

#### Description
Behave like *chash_lookup()* but only one randomly chosen target is returned among the *count* distinct targets.

#### Parameters
* *context*: pointer to an initialized context
* *name*: NULL-terminated candidate name
* *count*: desired targets count
* *output*: randomly chosen target (read-only value, *MUST* not be modified by calling code)

#### Return value
* *1*: when successful, count of returned matching targets (always 1 for this function)
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_INITIALIZED*: the context was not initialized (use *chash_initialize()* first)
* *CHASH_ERROR_NOT_FOUND*: no target exist in the given context (use *chash_add_target()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

PHP API
=======

General usage
-------------

The PHP API is more or less mapped onto and modelled after the C API (see above). An exact equivalent of the basic C example would be:

    $chash = new CHash();
    $chash->setTargets
    (
        array
        (
            'target001' => 1,
            'target002' => 1,
            'target003' => 1,
            'target004' => 1,
            'target005' => 1
        )
    );
    print $chash->lookupBalance('candidate001', 3) . "\n";

Error codes
-----------

The error codes from the C API are available verbatim the PHP extension. Because the initialization and termination of the C API context structure is performed by the PHP Zend object management functions, the *CHASH_ERROR_ALREADY_INITIALIZED* and *CHASH_ERROR_NOT_INITIALIZED* will never occur in the PHP extension. The list of the possible values and their meaning is provided below:

* *CHASH_ERROR_DONE* (0)
The function executed properly.

* *CHASH_ERROR_MEMORY* (-1)
A memory allocation error occurred.

* *CHASH_ERROR_IO* (-2)
A file I/O error occurred.

* *CHASH_ERROR_INVALID_PARAMETER* (-10)
An invalid parameter was provided.

* *CHASH_ERROR_NOT_FOUND* (-13)
The requested data cannot be found.

Exceptions
----------

In addition to methods returning error codes, exceptions are thrown by the PHP extension should an error occur. The list of the possible exceptions and their meaning is provided below (the original error code is stored in the exception code value):

* *CHashMemoryException*
A memory allocation error occurred.

* *CHashIOException*
A file I/O error occurred.

* *CHashInvalidParameterException*
An invalid parameter was provided.

* *CHashNotFoundException*
The requested data cannot be found.

Note: the exceptions mode is activated by default, use the *useExceptions()* method below to disable it.

Methods list
------------

* bool useExceptions(bool $use)

#### Description
Enable or disable exceptions mode.

#### Parameters
* *$use*: whether or not to use exceptions

#### Return value
* *true | false*: exceptions mode status

### int addTarget(string $target\[, int $weight\])

#### Description
Add a new target to the context. If the target already exists, its weight is updated according to the *weight* parameter.

#### Parameters
* *$target*: target name
* *$weight*: optional target weight (between 0 and 100, 1 if not specified)

#### Return value
* *CHASH_ERROR_DONE*: target was successfully added
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the method
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int removeTarget(string $target)

#### Description
Remove a target from the context.

#### Parameters
* *$target*: target name

#### Return value
* *CHASH_ERROR_DONE*: target was successfully removed
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the method|
* *CHASH_ERROR_NOT_FOUND*: the specified target does not exist in the given context

### int setTargets(array $targets)

#### Description
Behave like *addTarget()*, except multiple targets can be set in a single call and existing targets are discarded.

#### Parameters
* *$targets*: associative array of targets (keys are targets names, values are targets weights)

#### Return value
* *CHASH_ERROR_DONE*: target was successfully added
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the method
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int clearTargets()

#### Description
Clear all targets from the context.

#### Return value
* *CHASH_ERROR_DONE*: all targets were successfully removed

### int getTargetsCount()

#### Description
Return the number of targets in the context.

#### Return value
* *int*: number of targets

### string serialize()

#### Description
Serialize the context state into an opaque string (the context state can be restored by passing this string back to *unserialize()*).

#### Return value
* *string*: when successful, non empty string
* *''*: when not successful, empty string

### int unserialize(string $serialized)

#### Description
Restore a previously serialized context state (using *serialize()*).

#### Parameters
* *$serialized*: serialized data (generated using serialize())

#### Return value
* *int*: when successful, number of items in the newly restored continuum
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the method
* *CHASH_ERROR_NOT_FOUND*: no target exist in the serialized data (i.e. the serialized data is not coherent)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred

### int serializeToFile(string $path)

#### Description
Serialize the given context state into a file (the context state can be restored by passing this file path back to *unserializeFromFile()*).

#### Parameters
* *$path*: path of the file to which the context state will be saved

#### Return value
* *int*: when successful, size in bytes of the serialized data
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the method
* *CHASH_ERROR_NOT_FOUND*: no target exist in the given context (use *addTarget()* or *setTargets()* first)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred
* *CHASH_ERROR_IO*: an I/O error occurred (i.e. the given file path couldn't be written to)

### int unserializeFromFile(string $path)

#### Description
Restore a previously serialized context state (using *serializeToFile()*).

#### Parameters
* *$path*: path of the file from which the context state will be restored

#### Return value
* *int*: when successful, number of items in the newly restored continuum
* *CHASH_ERROR_INVALID_PARAMETER*: an invalid parameter was passed to the function
* *CHASH_ERROR_NOT_FOUND*: no target exist in the serialized data (i.e. the serialized data is not coherent)
* *CHASH_ERROR_MEMORY*: a memory allocation error occurred
* *CHASH_ERROR_IO*: an I/O error occurred (i.e. the given file path couldn't be read from)

### array lookupList(string $candidate\[, int $count\])

#### Description
Perform a lookup, returning *count* distincts targets.

#### Parameters
* *$candidate*: candidate name
* *$count*: desired targets count (1 if not specified)

#### Return value
* *array*: when successful, array of matching targets names
* *[]*: when not successful, empty array

### string lookupBalance(string $candidate\[, int $count\])

#### Description
Behave like *lookupList()* but only one randomly chosen target is returned among the *count* distinct targets.

#### Parameters
* *$candidate*: candidate name
* *$count*: desired targets count (1 if not specified)

#### Return value
* *string*: when successful, randomly chosen target name
* *''*: when not successful, empty string

Python API
----------

TODO
