========================================================
Proposed Protocols for Sparse ``ndrray`` Implementations
========================================================
I propose the name ``sarray`` for this (pronounced S-Array, abbreviation for Sparse ARRAY).
All protocols listed here are purely optional except where noted and projects using arrays that support this
interface will have to define what exactly the project or an operation requires.

Format-Agnostic Functions
=========================
Checking if an object is an ``sarray``
--------------------------------------

Implementation objects should have an attribute of ``__is_sarray__`` such that ``bool(object.__is_sarray__)``
evaluates to ``True`` if they implement this interface. Code to check if something is an ``sarray``
will be of the form. Only if the attribute exists and evaluates to ``True`` will the array be
considered an ``sarray``. Any errors during this conversion will be propagated up. ::


   if getattr(obj, '__is_sarray__', False):
       # Do something with sarray
   else:
       # Error handling code/do something else

Checking the format of an ``sarray``
------------------------------------
Implementation objects must have a ``format`` attribute that should be a Python string. It
should specify the *most specific* format of this array. For a description of the formats
supported by this set of protocols, read the section on formats. It is completely up to the
implementation which formats it supports. ::


   if obj.format == format:
       # Format-specific codes.

The format code is the lowercase abbreviation for the format in all cases.

Sub-formats do not necessarily need to implement all the things required by super-formats.
Super-formats are provided so that implementations can be unified; Nor do super-formats need
to handle all cases for sub-formats. However; if a certain format code is returned, then it
should implement everything that particular format requires.

A library must be able to convert between all formats it supports with an ``asformat()``
method which takes a single argument by default: the format code. Unsupported formats should
raise a ``ValueError``.

All formats must provide constructors that take all the mandatory array properties as a tuple in the first
argument, and all others as kwargs.

Getting a type for a specific format
------------------------------------
All ``sarray`` types must implement a static ``gettype(format)`` method, which will give the type for that
specific format for the given implementation. For example, the following code should return the type
for the CSR implementation. ::

   type(obj).gettype('csr')

Constructing a specific/generic format
--------------------------------------
The types of all formats must provide constructors that take all the mandatory array properties as a tuple in the first
argument, and all others as kwargs. For example, for CSR, both of the following would need to work. ::

   new_obj = type(obj)((data, indices, indptr), shape=shape) # If obj is CSR
   new_obj = type(obj).gettype('csr')((data, indices, indptr), shape=shape) # Arbitrary sarray type


Universally Supported Attributes
--------------------------------
All implementations must support the following ``ndarray`` attributes at minimum.

* `shape <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.shape.html>`_
* `ndim <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.ndim.html#numpy.ndarray.ndim>`_
* `dtype <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.dtype.html#numpy.ndarray.dtype>`_

Supported Formats
=================
CSR
---
See the `Scipy page on CSR <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html>`_.

This format is a sub-format of CSD and BSR.

Must provide at least the following extra attributes:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray`` objects.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the rows (``axis=ndim-2``) are uncompressed.

CSC
---
See the `Scipy page on CSC <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_matrix.html>`_.

This format is a sub-format of CSD and BSC.

Must provide at least the following extra attributes:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray`` objects.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the columns (``axis=ndim-1``) are uncompressed.

COO
---
See the `Scipy page on COO <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.coo_matrix.html>`_.

This format is a sub-format of CSD and BOO.

Must provide at least the following extra attributes:

* ``data``
* ``coords``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray`` objects.

``rows`` on the Scipy page corresponds to ``coords[-2]``  and ``cols`` to ``coords[-1]``.

``coords`` is a ``(ndim, nnz)`` shaped array that contains the coordinates of the nonzero elements.

.. _array_interface: https://docs.scipy.org/doc/numpy/reference/arrays.interface.html

CSD
---
An acronym for Compressed Sparse Dimensions. A generalization of CSR, CSC and COO.

This format is a sub-format of BSD.

* CSR is CSD with all axes compressed except ``ndim - 2``
* CSC is CSD with all axes compressed except ``ndim - 1``
* COO is CSD with no axes compressed.

Mandatory: CSD can store any number of non-compressed axes in ``coords`` and any number of compressed
axes in ``indptr`` (where these axes will be linearized before being compressed). Additionally,
it exposes an extra attribute, ``compressedaxes`` which lists the compressed axes *in order* in a ``tuple[int]``.
It also exposes ``data`` (same as above).

Optional: It should provide an ``indices`` attribute which must be ``coords[0]`` iff if ``len(compressed_axes) = 1``
and raise a ``ValueError`` otherwise.

``asformat`` will take an additional mandatory argument: ``compressedaxes``.

BSR, BSC, BOO, and BSD
----------------------
These acronyms aren't (strictly speaking) correct, but they are keeping in line with current
conventions.

See `Scipy page on BSR <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.bsr_matrix.html>`_.

They represent Block Compressed Row, Block Compressed Column, Block Coordinate and Block Compressed
Dimensions respectively. An implementation can implement any combination of these it so chooses.

CSR, CSC, COO, and CSD are sub-formats of these for a block size of ``(1,) * ndim``.

Mandatory: The only difference with the above is that certain dimensions are in blocks.
``data`` in this case is a ``(nnz_blocks * block_size)`` shaped array.

``coords``, ``indices``, ``indptr`` should all be divided by the block size where appropriate
so they address blocks and not elements.

It also provides a ``blocksize`` attribute, which is ``tuple[int] (ndim,)``.

``asformat`` will take an additional optional argument: ``blocksize``, along with any arguments
required for sub-formats. By default, the block size will not be changed on conversion.

Optional: It should provide a ``blockdata`` attribute which will be simply ``data.reshape((-1,) +
blocksize)``.

Block formats must provide a ``__is_bsparse__`` (abbreviation for Is Block Sparse) attribute that
checks for block format storage. If the returned format is non-block, this must also evaluate to
``False`` or not be present.

DOK
---
DOK is a read-write format by default. It must implement ``__getitem__`` and ``__setitem__`` for
individual items.

See the `Scipy page on DOK <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.dok_matrix.html>`_.

BDOK
----
BDOK is read-write, and supports  ``__getitem__`` and ``__setitem__`` for  values that only read from or
affect a single block respectively. It must also follow block matrix conventions. This is a super format of
DOK.

LIL
---
LIL is a write-only format by default, although implementations can implement reads if they so wish.
It must implement ``__setitem__`` such that if ``__setitem__`` can only be called in succession with
C-ordered indices.

See the `Scipy page on LIL <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.lil_matrix.html>`_.

BLIL
----
LIL is a write-only format by default, although implementations can implement reads if they so wish.
It must implement ``__setitem__`` such that if ``__setitem__`` can only be called in succession with
C-ordered indices of blocks. It must also follow block matrix conventions. This is a super-format of
LIL.

DIA
---
DIA must have the following additonal properties:

* ``data``
* ``offsets``

See the `Scipy page on DIA <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.dia_matrix.html>`_.

BDIA
----
The block extension for DIA. ``data`` must be of the size ``(number_of_blocks_in_main_diagonal * block_size)``.
Must follow block format conventions.
