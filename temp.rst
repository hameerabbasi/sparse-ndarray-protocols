.. notes
   * ask library maintainers about
      * Can you look our proposal over?
      * anecdotal evidence of experience with minimize
      * How would this SciPy enhancement proposal currently help your library?
   * If this had been present when development of your library began, how would have it influenced your library?
   * Libraries: Dask, XArray, Dask-ML, scikit-learn, Pysparse, others.

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
      * Other:
         * scikit.optimization (class based, no webpage (download from PyPI)).
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
`spmatrix` subclasses. This will be a three-step process:

- Make `spmatrix` subclasses adhere to the interface proposed below.
- Rewrite parts of scipy taking in `spmatrix` subclasses as input to
  accept anything following the interface.
- Write `sparray` and subclasses to follow the `ndarray` interface.

After these three steps are complete, the current `spmatrix` classes will be
deprecated and `sparray` will be made the new default. Alternatively, it will
be possible to make `spmatrix` a subclass of `sparray` with `operator.mul` just
performing matrix multiplication.

The newly-developed classes will:

- have easier maintenance, testing, and development.
- Follow the `ndarray` interface.

Sparse matrices are a critical part of SciPy's API, and we believe we can enhance
the API while retaining backwards compatibility and improving maintainability.

We propose introducing a unified interface for sparse arrays, which encourages
alternate implementations and promotes inter-operability. Old classes will be
deprecated, as `np.matrix` has been deprecated to be removed. If concise matrix
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


