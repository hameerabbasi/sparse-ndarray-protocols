.. notes
   * ask library maintainers about
      * Can you look our proposal over?
      * anecdotal evidence of experience with minimize
      * How would this SciPy enhancement proposal currently help your library?
   * If this had been present when development of your library began, how would have it influenced your library?
   * Libraries: Dask, XArray, Dask-ML, scikit-learn, Pysparse, others.

========================================================
Scientific PEP -- Protocols for Sparse `ndarray` Formats
========================================================

.. outline
   * Abstract
   * Introduction
      * Here's what a sparse array is...
         * It's a form of storing an array.
         * Usually arrays with lots of zero elements.
         * Point to many uses in scientific computation and ML.
      * Point to users of...
         * Sparse arrays in general
         * `scipy.sparse` (search GH issues)
   * Proposed solution
      * Proposed Interface
         * ``__is_sparray__`` - Check if array is sparse.
         * ``format`` - The storage format.
         * ``asformat(format)`` - Convert between formats.
         * Format-specific properties
      * Goals:
         * Allow for alternate implementations of sparse arrays
         * API cleaning and maintainability of ``scipy.sparse``
         * preserving backwards compatibility
         * exposing a new API to easily create sparse array formats
      * Examples
   * Goals
      * Allow for alternate implementations of sparse arrays
         * Have to explain why alternate implementations will be useful.
      * preserving backwards compatibility
         * Make `spmatrix` subclasses follow the interface
         * Long-term: Follow the `ndarray` interface.
      * allow alternate implementations
         * Provide standard interface for alternate implementations
         * So they can be plugged into Scipy methods and "just work"
      * cleaning the existing API
         * ``spmatrix`` and ``sparray`` can have shared code.
         * Alternate implementations can use the code as well.
         * Unify implementations where possible (CSR/CSC/COO -> CSD)
         * End fragmentation
   * Existing work
      * Point to MATLAB support for sparse arrays.
      * pydata/sparse
      * PySparse (not ideal)
   * Concerns
      * `sparray` classes should have a unified protocol.
      * Current `spmatrix` classes follow `np.matrix` which is not ideal.
      * Bugs in current implementation.
   * Open bugs
      * Search in PySparse, scipy.sparse, other scikit-learn, etc.
   * Implementation
      * List functions, attributes in more depth
      * Scope
      * Existing code (Point to pydata/sparse)
         * How would it work with C/Fortran optimizers?
         * What interface are we proposing? See proposed code below

*Abstract*

Proposal
========

We propose a gradual rewrite of the current functions taking in as input the
``spmatrix`` subclasses. This will be a three-step process:

- Make ``spmatrix`` subclasses adhere to the interface proposed below.
- Rewrite parts of scipy taking in ``spmatrix`` subclasses as input to
  accept anything following the interface.
- Write ``sparray`` and subclasses to follow the ``ndarray`` interface.

After these three steps are complete, the current ``spmatrix`` classes will be
deprecated and ``sparray`` will be made the new default. Alternatively, it will
be possible to make ``spmatrix`` a subclass of ``sparray`` with ``*`` just
performing matrix multiplication, and other ``np.matrix`` semantics.

The newly-developed classes will

- have easier maintenance, testing, and development,
- and follow the ``ndarray`` interface.

Sparse matrices are a critical part of SciPy's API, and we believe we can enhance
the API while retaining backwards compatibility and improving maintainability.

We propose introducing a unified interface for sparse arrays, which encourages
alternate implementations and promotes inter-operability. Old classes will be
deprecated, as ``np.matrix`` has been deprecated to be removed. If concise matrix
multiplication is needed, the matrix multiplication operator ``@`` will be
recommended.

Alternate implementations hoping to maintain inter-compatibility with SciPy
will adopt this interface, and be able to use SciPy methods on these arrays.

The proposed interface will provide

- a way to check if something is a sparse array,
- a way to detect its format,
- inter-format conversions,
- standard constructors,
- a shape attribute,
- and format specific attributes.

Existing Work
-------------

* There is some work ongoing in `pydata/sparse <https://github.com/pydata/sparse>`_ to make this a reality.
* `MATLAB supports sparse arrays <https://de.mathworks.com/help/matlab/ref/sparse.htm>`_.
* `An implementation of COO by CJ Carey <https://github.com/perimosocordiae/sparray>`_
* `An implementation of DOK by Evgeni Burovski <https://github.com/ev-br/sparr>`_
* `An implementation by Nick R. Papior <https://github.com/zerothi/sisl/blob/master/sisl/sparse.py>`_

Open Issues
-----------

* `scipy/scipy#8162 <https://github.com/scipy/scipy/issues/8162>`_
* `dask/dask-ml#123 <https://github.com/dask/dask-ml/issues/123>`_
* `gammapy/gammapy#1332 <https://github.com/gammapy/gammapy/issues/1332>`_
* `pangeo-data/pangeo#120 <https://github.com/pangeo-data/pangeo/issues/120>`_
* `TileDB-Inc/TileDB-Py#23 <https://github.com/TileDB-Inc/TileDB-Py/issues/23>`_

.. note
   Find more issues to add here.

Enhancements
============

Alternative Implementations
---------------------------

If these protocols are followed; it is well within reach to make ``scipy.sparse`` work
with any alternative implementations, even returning structures from the original
implementation.

Under our proposal, the workflow for any Scipy Operation would look like the following:

* Check if the input is a sparse array
* (Optional) Convert to appropriate format from same implementation.
* Work with format natively.
* Call the constructor from the specific implementation at the end.

Format-Agnostic Functions
-------------------------

Checking if an object is a sparse array
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation objects should have an attribute of ``__is_sparray__`` such that ``bool(object.__is_sparray__)``
evaluates to ``True`` if they implement this interface. Code to check if something is a sparse array
will be of the form. Only if the attribute exists and evaluates to ``True`` will the array be
considered an sparse array. Any errors during this conversion will be propagated up. The code to check if
something is a sparse array will look like the following:

.. code-block:: python

   if getattr(obj, '__is_sparray__', None):
       # Do something with sparse array
   else:
       # Error handling code/do something else

Universally Supported Attributes
--------------------------------

All implementations must support the following ``ndarray`` attributes at minimum.

* `shape <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.shape.html>`_
* `size <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.size.html>`_
* `ndim <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.ndim.html>`_
* `dtype <https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.dtype.html>`_
* __len__

Checking the Format of a Sparse Array
-------------------------------------

Implementation objects must have a ``format`` attribute that should be a Python string. It
should specify the *most specific* format of this array. For a description of the formats
supported by this set of protocols, read the section on formats. It is completely up to the
implementation which formats it supports.

.. code-block:: python

   if obj.format == 'csr':
       # CSR-specific code.

The format code is the lowercase abbreviation for the format in all cases.

Sub-formats do not necessarily need to implement all the things required by super-formats;
nor do super-formats need to handle all cases for sub-formats. Super-formats are provided so
that implementations can be unified; However; if a certain format code is returned, then it
should implement everything that particular format requires.

Converting Between Formats
--------------------------

A library must be able to convert between all formats it supports with an ``asformat()``
method which takes a single argument by default: the format code. Unsupported formats should
return ``NotImplemented``. The code to convert to a format will look like the following:

.. code-block:: python

   if obj.format != 'csr':
       obj = obj.asformat('csr')

   # Do something with CSR

Getting the type of a format
----------------------------

All types of sparse arrays must implement a ``gettype(format)`` method that would take in the
format code, and return the type that supports all operations of that format. For example, to
get the type relating to CSR, both of the following code would work:

.. code-block:: python

   obj.gettype('csr') # Returns the CSR type for a given implementation.
   type(obj).gettype('csr') # Should be a class/static method.

Standard Constructors
---------------------

All formats must provide constructors that take all the mandatory array properties for a particular
format as a tuple in the first argument, and ``shape`` as a kwarg. If it's a format supporting writes,
``dtype`` should also be supported as a kwarg, otherwise, it will be inferred from the data. For example,
for CSR, the constructor would look like the following:

.. code-block:: python

   csr_object = csr_type((data, indices, indptr), shape=shape)

And for DOK, the following should work:

.. code-block:: python

   dok_object = dok_type(dtype=dtype, shape=shape)

Proposed Formats
----------------

CSR
^^^

See the `Scipy page on CSR <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html>`_.

This format is a sub-format of CSD and BSR.

Must provide at least the following extra attributes:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray`` objects.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the rows (``axis=ndim-2``) are compressed.

CSC
^^^

See the `Scipy page on CSC <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_matrix.html>`_.

This format is a sub-format of CSD and BSC.

Must provide at least the following extra attributes:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray`` objects.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the columns (``axis=ndim-1``) are compressed.

COO
^^^

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
^^^

An acronym for Compressed Sparse Dimensions. A generalization of CSR, CSC and COO.

This format is a sub-format of BSD.

* CSR is CSD with only rows (``ndim - 2``) compressed.
* CSC is CSD with only columns (``ndim - 1``) compressed.
* COO is CSD with no axes compressed.

Mandatory: CSD can store any number of non-compressed axes in ``coords`` and any number of compressed
axes in ``indptr`` (where these axes will be linearized before being compressed). Additionally,
it exposes an extra attribute, ``compressedaxes`` which lists the compressed axes *in order* in a ``tuple[int]``.
It also exposes ``data`` (same as above).

Optional: It should provide an ``indices`` attribute which must be ``coords[0]`` iff if ``len(compressed_axes) = 1``
and raise a ``ValueError`` otherwise.

``asformat`` will take an additional mandatory argument: ``compressedaxes``.

BSR, BSC, BOO, and BSD
^^^^^^^^^^^^^^^^^^^^^^

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
^^^

DOK is a read-write format by default. It must implement ``__getitem__`` and ``__setitem__`` for
individual items.

See the `Scipy page on DOK <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.dok_matrix.html>`_.

BDOK
^^^^

BDOK is read-write, and supports  ``__getitem__`` and ``__setitem__`` for  values that only read from or
affect a single block respectively. It must also follow block matrix conventions. This is a super format of
DOK.

LIL
^^^

LIL is a write-only format by default, although implementations can implement reads if they so wish.
It must implement ``__setitem__`` such that if ``__setitem__`` can only be called in succession with
C-ordered indices.

See the `Scipy page on LIL <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.lil_matrix.html>`_.

BLIL
^^^^

BLIL is a write-only format by default, although implementations can implement reads if they so wish.
It must implement ``__setitem__`` such that if ``__setitem__`` can only be called in succession with
C-ordered indices of blocks. It must also follow block matrix conventions. This is a super-format of
LIL.

DIA
^^^

DIA must have the following additonal properties:

* ``data``
* ``offsets``

See the `Scipy page on DIA <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.dia_matrix.html>`_.

BDIA
^^^^

The block extension for DIA. ``data`` must be of the shape ``(number_of_blocks_in_main_diagonal * block_size,)``.
Must follow block format conventions.
