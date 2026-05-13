.. _ch.gen.madx:

MADX
====

The keyword :literal:`MADX` denotes the special environment within MAD-NG that emulates the behavior of the global workspace of MAD-X. It is a compatibility workspace for importing MAD-X and MAD8 lattice descriptions into MAD-NG.

Environment
-----------

The object :literal:`MADX` is available globally. Its mutable child is used as the working environment when MAD-X or MAD8 files are imported, so names defined by these files become variables, elements, sequences or beam lines inside :literal:`MADX`.

:literal:`MADX` is not the CERN MAD-X program embedded into MAD-NG, and it does not execute MAD-X code directly. Instead, MAD-X or MAD8 input is translated to MAD-NG code and then loaded into this environment. The resulting objects are ordinary MAD-NG objects that can be inspected, modified and used by MAD-NG commands such as :var:`track`, :var:`twiss`, :var:`cofind` or :var:`match`.

In practice, :literal:`MADX` is the workspace that stores imported legacy names. After loading a sequence or optics deck, users typically:

* access imported objects directly as :literal:`MADX.ps`, :literal:`MADX.lhcb1`, :literal:`MADX.nrj`, ...
* bring selected names into local scope with :literal:`local ps, nrj in MADX`
* update imported knobs and deferred expressions through :literal:`MADX`

Typical workflow:

.. code-block:: lua

   MADX:load("fodo.seq", "fodo.mad")
   local seq in MADX
   seq.beam = beam
   mtbl, mflw = twiss { sequence=seq, implicit=true, nslice=10 }

The environment exposes the option set :literal:`MADX.option` with the following defaults:

**debug**
   Enable extra diagnostic output during translation and loading. (default: :const:`false`)

**info**
   Enable informational output during translation and loading. (default: :const:`false`)

**warn**
   Emit warnings for compatibility fallbacks, in particular when an undefined MAD-X name is read and auto-created with value :const:`0`. (default: :const:`true`)

**rbarc**
   Interpret RBEND lengths using the MAD-X arc/chord compatibility rules during import. (default: :const:`true`)

**nopub**
   Prevent imported sequence elements from being published automatically in the :literal:`MADX` environment. (default: :const:`false`)

Methods
-------

**load**
   A *method* :literal:`MADX:load(src [, dst [, cfg]])` that loads MAD-X, MAD8 or already translated MAD-NG input.

   If :literal:`src` is a MAD-X/MAD8 file, it is translated to MAD-NG code and then loaded into the environment. If :literal:`dst` is provided, the translated MAD-NG file is written there with the :literal:`.mad` suffix before being loaded. If :literal:`src` is already a :literal:`.mad` file and :literal:`dst` is omitted, it is compiled directly with no translation step.

   This is the standard entry point to import legacy MAD-X or MAD8 models. It is common to call :literal:`MADX:load(...)` several times in a row to import a base sequence, one or more optics files, and optional helper files that predefine otherwise unset MAD-X variables.

   The optional configuration table :literal:`cfg` currently supports:

   * :literal:`reload` to force regeneration of :literal:`dst` even when it is newer than :literal:`src`
   * :literal:`mad8` to enable MAD8 parsing rules
   * :literal:`rbarc` to override :literal:`MADX.option.rbarc` for the current load
   * :literal:`nopub` to override :literal:`MADX.option.nopub` for the current load

**open_env**
   A *method* :literal:`MADX:open_env([ctx])` that opens the MADX object as the active object environment and sets :literal:`option.madxenv` accordingly.

   This is mainly useful when one wants to evaluate a block of assignments using bare MAD-X names instead of prefixed accesses through :literal:`MADX`. For example:

   .. code-block:: lua

      MADX:open_env()
      dqx_b1 = 6.65239e-4
      dqy_b1 = 3.31034e-3
      MADX:close_env()

**close_env**
   A *method* :literal:`MADX:close_env()` that closes the active MADX object environment and clears :literal:`option.madxenv`.

Functions and Constants
-----------------------

The :literal:`MADX` environment is preloaded with:

* all variables from :mod:`MAD.constant`
* all element classes from :mod:`MAD.element`
* MAD-X-like scalar helpers such as :literal:`abs`, :literal:`sqrt`, :literal:`exp`, :literal:`log`, :literal:`sin`, :literal:`cos`, :literal:`tan`, :literal:`asin`, :literal:`acos`, :literal:`atan`, :literal:`erf`, :literal:`erfc`, :literal:`fact`, :literal:`sinc`, :literal:`round` and :literal:`frac`
* random helpers :literal:`ranf()`, :literal:`gauss()`, :literal:`tgauss(cut)`, :literal:`poisson(lambda)` and :literal:`seed(seed)`
* geometry helpers :literal:`cord2arc`, :literal:`arc2cord`, :literal:`cord2arc_w` and :literal:`arc2cord_w`

Compatibility Aliases
---------------------

The environment also defines a few MAD-X-specific aliases and compatibility objects:

* :literal:`MADX.multipole` overrides the survey angle convention so that :literal:`knl[1]` contributes like MAD-X
* :literal:`MADX.rcollimator`, :literal:`MADX.ecollimator`, :literal:`MADX.translation` and :literal:`MADX.elseparator` are provided as aliases not present in the base MAD element namespace
* string identifiers such as :literal:`centre`, :literal:`circle`, :literal:`ellipse`, :literal:`rectellipse`, :literal:`entry` and :literal:`exit` are preloaded, with :literal:`center` mapped to :literal:`centre`

Names and Lookup
----------------

MAD-X names are normalized before lookup:

* names are converted to lower case
* :literal:`.` and :literal:`$` are converted to :literal:`_`
* repeated primes are rewritten to suffixes such as :literal:`_prime`, :literal:`_2prime`, and so on

Reading an undefined name from :literal:`MADX` creates it lazily with value :const:`0`. If :literal:`MADX.option.warn == true`, a warning is emitted before the zero value is stored.

The same imported object can therefore be reached in several equivalent ways. For example, after importing a sequence named :literal:`ps`, one may use:

* :literal:`MADX.ps`
* :literal:`local ps in MADX`
