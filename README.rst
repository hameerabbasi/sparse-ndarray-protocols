========================================================
Proposed Protocols for Sparse ``ndrray`` Implementations
========================================================
I propose the name ``sarray`` for this (pronounced S-Array, abbreviation for Sparse ARRAY).
All protocols listed here are purely optional except where noted and projects using arrays that support this
interface will have to define what exactly the project or an operation requires.

Format-Agnostic Functions
=========================
.. rubric:: Checking if an object is an ``sarray``

Implementation objects should have an attribute of ``__is_sarray__`` such that ``bool(object.__is_sarray__)``
evaluates to ``True`` if they implement this interface. Code to check if something is an ``sarray``
will be of the form. Only if the attribute exists and evaluates to ``True`` will the array be
considered an ``sarray``. Any errors during this conversion will be propagated up.

.. code-block:: python
   if getattr(obj, '__is_sarray__', False):
       # Do something with sarray
   else:
       # Error handling code/do something else

.. rubric:: Checking the format of an ``sarray``
Implementation objects must have a ``format`` attribute that should be a Python string. It
should specify the *most specific* format of this array. For a description of the formats
supported by this set of protocols, read the section on formats. It is completely up to the
implementation which formats it supports.

.. code-block:: python
   if getattr(obj, 'format', None) == format:
       # Format-specific codes.

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
Format code is ``csr``. See the `Scipy page on CSR <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html>`_.

Must provide at least the following extra properties:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray``s.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the columns (``axis=ndim-1``) are uncompressed.

Optionally, implementations can provide a ``tocsr()`` method to convert the array to CSR.

CSC
---
Format code is ``csc``. See the `Scipy page on CSC <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_matrix.html>`_.

Must provide at least the following extra properties:

* ``data``
* ``indices``
* ``indptr``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray``s.

In line with the SciPy conventions for CSR, but with the following exception: If ``ndim > 2`` is supported, then
CSD conventions are followed where *only* the rows (``axis=ndim-2``) are uncompressed.

Optionally, implementations can provide a ``tocoo()`` method to convert the array to CSR.

COO
---
Format code is ``coo``. See the `Scipy page on COO <https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.coo_matrix.html>`_.

Must provide at least the following extra properties:

* ``data``
* ``coords``

All of these must follow the `array interface <array_interface>`_, but do not need to be ``ndarray``s.

``rows`` on the Scipy page corresponds to ``coords[-2]``  and ``cols`` to ``coords[-1]``.

``coords`` is a ``(ndim, nnz)`` shaped array that contains the coordinates of the nonzero elements.

.. _array_interface: https://docs.scipy.org/doc/numpy/reference/arrays.interface.html
