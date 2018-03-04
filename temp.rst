.. notes
   * ask library maintainers about
      * Can you look our proposal over?
      * anecdotal evidence of experience with minimize
      * How would this SciPy enhancement proposal currently help your library?
   * If this had been present when development of your library began, how would have it influenced your library?
   * Libraries: Dask, XArray, Dask-ML, scikit-learn, others.

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
