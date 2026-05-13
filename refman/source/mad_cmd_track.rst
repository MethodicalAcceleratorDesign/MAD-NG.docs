Track
=====
.. _ch.cmd.track:

The :var:`track` command provides a simple interface to the *dynamic* tracking code. [#f1]_ The dynamic tracking can be used to track the particles in the :ref:`local reference system <sec.phy.lrs>` while running through the elements of a sequence. The particles coordinates can be expressed in the :ref:`global reference system <sec.phy.grs>` by changing from the local to the global frames using the information delivered by the :doc:`survey <mad_cmd_survey>` command.


.. code-block:: lua
    :caption: Synopsis of the :var:`track` command with default setup.
    :name: fig-track-synop

    mtbl, mflw [, eidx] = track {
        sequence=sequ,   -- sequence (required)
        beam=nil,        -- beam (or sequence.beam, required)
        range=nil,       -- range of tracking (or sequence.range)
        dir=1,           -- s-direction of tracking (1 or -1)
        s0=0,            -- initial s-position offset [m]
        X0=0,            -- initial coordinates (or damap(s), or beta block(s))
        O0=0,            -- initial coordinates of reference orbit
        deltap=0,        -- initial deltap(s)
        nturn=1,         -- number of turns to track
        nstep=-1,        -- number of elements to track
        nslice=1,        -- number of slices (or weights) for each element
        method=4,        -- method or order for integration (1 to 8)
        model='TKT',     -- model for integration ('DKD' or 'TKT')
        mapdef=false,    -- setup for damap (or list of, true => {})
        secnmul=false,   -- curved-multipole expansion for sbends
        implicit=false,  -- slice implicit elements too (e.g. plots)
        misalign=false,  -- consider misalignments
        aperture=false,  -- default aperture
        fringe=true,     -- enable fringe fields (see element.flags.fringe)
        frngmax=2,       -- maximum multipole fringe field order
        radiate=false,   -- radiate at slices
        taper=false,     -- tapering compensation
        nocavity=false,  -- disable rfcavities
        totalpath=false, -- variable 't' is the totalpath
        cmap=true,       -- use C/C++ maps when available
        ptcmodel=nil,    -- nil=MADNG, true=PTC, false=MADX
        save=true,       -- create mtable and save results
        aper=true,       -- check for aperture
        observe=1,       -- save only in observed elements (every n turns)
        reserve=0,       -- preallocate output rows (0 => guess)
        savesel=fnil,    -- save selector (predicate)
        savemap=false,   -- save damap in the column __map
        atentry=fnil,    -- action called when entering an element
        atslice=fnil,    -- action called after each element slices
        atexit=fnil,     -- action called when exiting an element
        atsave=fnil,     -- action called when saving in mtable
        ataper=fnil,     -- action called when checking for aperture
        atdebug=fnil,    -- action called when debugging the element maps
        apersel=fnil,    -- aperture selector (predicate)
        coitr=nil,       -- closed-orbit iteration limit
        cotol=nil,       -- closed-orbit tolerance
        costp=nil,       -- finite-difference scale for closed-orbit Jacobian
        O1=nil,          -- optional final closed-orbit translation
        info=nil,        -- information level (output on terminal)
        debug=nil,       -- debug information level (output on terminal)
        usrdef=nil,      -- user defined data attached to the mflow
        mflow=nil,       -- mflow, exclusive with other attributes except nstep
    }

.. _sec.track.synop:

Command synopsis
----------------

| The :var:`track` command format is summarized in :numref:`fig-track-synop`, including the default setup of the attributes.
| The :var:`track` command supports the following attributes:

**sequence**
    The *sequence* to track. (no default, required).

    Example: :expr:`sequence = lhcb1`.

**beam**
    The reference *beam* for the tracking. If no beam is provided, the command looks for a beam attached to the sequence, i.e. the attribute :literal:`seq.beam`. [#f2]_ (default: :const:`nil`).

    Example: :expr:`beam = beam 'lhcbeam' { ... }` where ... are the *beam-attributes*.

**range**
    A *range* specifying the span of the sequence track. If no range is provided, the command looks for a range attached to the sequence, i.e. the attribute :attr:`seq.range`. (default: :const:`nil`).

    Example: :expr:`range = "S.DS.L8.B1/E.DS.R8.B1"`.

**dir**
    The :math:`s`-direction of the tracking: :const:`1` forward, :const:`-1` backward. (default: 1).

    Example: :expr:`dir = -1`.

**s0**
    A *number* specifying the initial :math:`s`-position offset. (default: :math:`0` [m]).

    Example: :expr:`s0 = 5000`.

**X0**
    A *mappable* (or a list of *mappable*) specifying initial coordinates :literal:`{x,px,y,py,t,pt}`, damap, or beta block for each tracked object, i.e. particle or damap. The beta blocks are converted to damaps, while the coordinates are converted to damaps only if :literal:`mapdef` is specified, but both will use :literal:`mapdef` to setup the damap constructor. Each tracked object may also contain a :var:`beam` to override the reference beam, and a *logical* :literal:`nosave` to discard this object from being saved in the mtable. (default: 0).

    Example: :expr:`X0 = { x=1e-3, px=-1e-5 }`.

**O0**
    A *mappable* specifying initial coordinates :literal:`{x,px,y,py,t,pt}` of the reference orbit around which X0 definitions take place. If it has the attribute :expr:`cofind == true`, it will be used as an initial guess to search for the reference closed orbit. (default: 0).

    Example: :expr:`O0 = { x=1e-4, px=-2e-5, y=-2e-4, py=1e-5 }`.

**deltap**
    A *number* (or list of *number*) specifying the initial :math:`\delta_p` to convert (using the beam) and add to the :literal:`pt` of each tracked particle or damap. (default: :const:`0`).

    Example: :expr:`s0 = 5000`.

**nturn**
    A *number* specifying the number of turn to track. (default: 1).

    Example: :expr:`nturn = 2`.

**nstep**
    A *number* specifying the number of element to track. A negative value will track all elements. (default: -1).

    Example: :expr:`nstep = 1`.

**nslice**
    A *number* specifying the number of slices or an *iterable* of increasing relative positions or a *callable* :literal:`(elm, mflw, lw)` returning one of the two previous kind of positions to track in the elements. The arguments of the callable are in order, the current element, the tracked map flow, and the length weight of the step. This attribute can be locally overridden by the element. (default: 1).

    Example: :expr:`nslice = 5`.

**mapdef**
    A *logical* or a *damap* specification as defined by the :doc:`DAmap <mad_mod_diffmap>` module to track DA maps instead of particles coordinates. A value of :const:`true` is equivalent to invoke the *damap* constructor with :literal:`{}` as argument. (default: :const:`false`).

    Example: :expr:`mapdef = { xy=2, pt=5 }`.

**method**
    A *number* specifying the order of integration from 1 to 8, or a *string* specifying a special method of integration. Odd orders are rounded to the next even order to select the corresponding Yoshida or Boole integration schemes. The special methods are :literal:`simple` (equiv. to :literal:`DKD` order 2), :literal:`collim` (equiv. to :literal:`MKM` order 2), and :literal:`teapot` (Teapot splitting order 2). (default: 4).

    Example: :expr:`method = 'teapot'`.

**model**
    A *string* specifying the integration model, either :literal:`'DKD'` for *Drift-Kick-Drift* thin lens integration or :literal:`'TKT'` for *Thick-Kick-Thick* thick lens integration. [#f3]_ (default: :literal:`'TKT'`)

    Example: :expr:`model = 'DKD'`.

**secnmul**
    A *logical* or a non-negative *number* controlling the curved-multipole expansion used for SBEND thick maps. A false value disables the extra curved-multipole terms. (default: :const:`false`).

    Example: :expr:`secnmul = 6`.

**ptcmodel**
    A *logical* selecting the tracking backend: :const:`nil` keeps the MAD-NG model, :const:`true` selects the strict PTC model, and :const:`false` selects the MAD-X-compatible model. [#f7]_ (default: :const:`nil`)

    Example: :expr:`ptcmodel = true`.

**implicit**
    A *logical* indicating that implicit elements must be sliced too, e.g. for smooth plotting. (default: :const:`false`).

    Example: :expr:`implicit = true`.

**misalign**
    A *logical* indicating that misalignment must be considered. (default: :const:`false`).

    Example: :expr:`misalign = true`.

**aperture**
    A *mappable* specifying the default aperture to consider when it is missing at element level, e.g. in implicit drift. A false value lets :var:`track` fall back to :literal:`sequence.aperture` or, if absent, to :expr:`{kind='circle', 1}`. (default: :const:`false`).

    Example: :expr:`aperture = {kind='square', 10}`.

**fringe**
    A *logical* indicating that fringe fields must be considered or a *number* specifying a bit mask to apply to all elements fringe flags defined by the element module. The value :const:`true` is equivalent to the bit mask , i.e. allow all elements (default) fringe fields. (default: :const:`true`).

    Example: :expr:`fringe = false`.

**frngmax**
    A non-negative *number* specifying the maximum multipole order used for fringe-field expansions. (default: :const:`2`).

    Example: :expr:`frngmax = 4`.

**radiate**
    A *logical* enabling radiation, or a *string* selecting the radiation mode :literal:`'damping'`, :literal:`'quantum'` or :literal:`'photon'`. A trailing :literal:`'+'` keeps the corresponding bookkeeping enabled, e.g. :literal:`'damping+'` or :literal:`'quantum+'`. The value :const:`true` is equivalent to damping radiation. (default: :const:`false`).

    Example: :expr:`radiate = 'quantum'`.

**taper**
    A *logical* or a positive *number* enabling tapering compensation. The value :const:`true` is converted to :literal:`3`, i.e. tapering is applied during the first three passes tracked by the map flow. Any positive number limits the compensation to that many passes. (default: :const:`false`).

    Example: :expr:`taper = true`.

**nocavity**
    A *logical* disabling RF cavities and therefore enforcing 5D tracking. (default: :const:`false`).

    Example: :expr:`nocavity = true`.

**totalpath**
    A *logical* indicating to use the totalpath for the fifth variable :literal:`'t'` instead of the local path. (default: :const:`false`).

    Example: :expr:`totalpath = true`.

**cmap**
    A *logical* indicating whether the C/C++ map implementations should be used when available. (default: :const:`true`).

    Example: :expr:`cmap = false`.

**save**
    A *logical* specifying to create a *mtable* and record tracking information at the observation points. The :literal:`save` attribute can also be a *string* specifying saving positions in the observed elements: :literal:`"atentry"`, :literal:`"atslice"`, :literal:`"atexit"` (i.e. :const:`true`), :literal:`"atbound"` (i.e. entry and exit), :literal:`"atbody"` (i.e. slices and exit) and :literal:`"atall"`. (default: :const:`true`).

    Example: :expr:`save = false`.

**aper**
    A *logical* or a *string* enabling the default aperture check attached to :literal:`ataper`. The value :const:`true` applies the check at the last slice. A *string* can be used to select the checking positions with the same directives as :literal:`save`, i.e. :literal:`"atentry"`, :literal:`"atslice"`, :literal:`"atexit"`, :literal:`"atbound"`, :literal:`"atbody"` or :literal:`"atall"`. (default: :const:`true`).

    Example: :expr:`aper = false`.

**observe**
    A *number* specifying the observation points to consider for recording the tracking information. A zero value will consider all elements, while a positive value will consider selected elements only, checked with method :meth:`:is_observed`, every :literal:`observe` :math:`>0` turns. (default: :const:`1` ).

    Example: :expr:`observe = 1`.

**reserve**
    A non-negative *number* used to preallocate the number of output rows in the produced *mtable*. A value of :const:`0` keeps the built-in size estimate. (default: :const:`0`).

    Example: :expr:`reserve = 50000`.

**savesel**
    A *callable* :literal:`(elm, mflw, lw, islc)` acting as a predicate on selected elements for observation, i.e. the element is discarded if the predicate returns :const:`false`. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`)

    Example: :expr:`savesel = \\e -> mylist[e.name] ~= nil`.

**savemap**
    A *logical* indicating to save the damap in the column :literal:`__map` of the *mtable*. (default: :const:`false`).

    Example: :expr:`savemap = true`.

**atentry**
    A *callable* :literal:`(elm, mflw, 0, -1)` invoked at element entry. The arguments are in order, the current element, the tracked map flow, zero length and the slice index . (default: :literal:`fnil`).

    Example: :expr:`atentry = myaction`.

**atslice**
    A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element slice. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`).

    Example: :expr:`atslice = myaction`.

**atexit**
    A *callable* :literal:`(elm, mflw, 0, -2)` invoked at element exit. The arguments are in order, the current element, the tracked map flow, zero length and the slice index . (default: :literal:`fnil`).

    Example: :expr:`atexit = myaction`.

**ataper**
    A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element aperture checks, by default at last slice. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. If a particle or a damap hits the aperture, then its :literal:`status = "lost"` and it is removed from the list of tracked items. (default: :literal:`fnil`).

    Example: :expr:`ataper = myaction`.

**atsave**
    A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element saving steps, by default at exit. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`).

    Example: :expr:`atsave = myaction`.

**atdebug**
    A *callable* :literal:`(elm, mflw, lw, [msg], [...])` invoked at the entry and exit of element maps during the integration steps, i.e. within the slices. The arguments are in order, the current element, the tracked map flow, the length weight of the integration step and a *string* specifying a debugging message, e.g. :literal:`"map_name:0"` for entry and :literal:`":1"` for exit. If the level :literal:`debug` :math:`\geq 4` and :literal:`atdebug` is not specified, the default *function* :literal:`mdump` is used. In some cases, extra arguments could be passed to the method. (default: :literal:`fnil`).

    Example: :expr:`atdebug = myaction`.

**apersel**
    A *callable* :literal:`(elm, mflw, lw, islc)` acting as a predicate on aperture-check positions. If it returns :const:`false`, the default aperture action is skipped for that location. (default: :literal:`fnil`).

    Example: :expr:`apersel = \\e -> e.kind ~= "marker"`.

**coitr**
    A *number* specifying the maximum number of closed-orbit iterations when the map flow requests a closed-orbit search. (default: :const:`nil` and inherited from the closed-orbit helper).

    Example: :expr:`coitr = 25`.

**cotol**
    A *number* specifying the closed-orbit tolerance used when the map flow requests a closed-orbit search. (default: :const:`nil` and inherited from the closed-orbit helper).

    Example: :expr:`cotol = 1e-8`.

**costp**
    A *number* specifying the relative finite-difference scale used for the closed-orbit Jacobian when a closed-orbit search is requested. (default: :const:`nil` and inherited from the closed-orbit helper).

    Example: :expr:`costp = 1e-8`.

**O1**
    A *mappable* specifying the coordinates :literal:`{x,px,y,py,t,pt}` to subtract from the final closed-orbit coordinates when the closed-orbit helper is used. (default: :const:`nil`).

    Example: :expr:`O1 = { t=100, pt=10 }`.

**info**
    A *number* specifying the information level to control the verbosity of the output on the console. (default: :const:`nil`).

    Example: :expr:`info = 2`.

**debug**
    A *number* specifying the debug level to perform extra assertions and to control the verbosity of the output on the console. (default: :const:`nil`).

    Example: :expr:`debug = 2`.

**usrdef**
    Any user defined data that will be attached to the tracked map flow, which is internally passed to the elements method :meth:`:track` and to their underlying maps. (default: :const:`nil`).

    Example: :expr:`usrdef = { myvar=somevalue }`.

**mflow**
    An *mflow* containing the current state of a :var:`track` command. If a map flow is provided, the setup stored in the mflow is reused; in practice, only the volatile fields :literal:`nstep`, :literal:`info` and :literal:`debug` are refreshed by the command wrapper. (default: :const:`nil`).

    Example: :expr:`mflow = mflow0`.


The :var:`track` command returns the following objects in this order:

**mtbl**
    An *mtable* corresponding to the TFS table of the :var:`track` command.

**mflw**
    An *mflow* corresponding to the map flow of the :var:`track` command.

**eidx**
    An optional *number* corresponding to the last tracked element index in the sequence when :literal:`nstep` was specified and stopped the command before the end of the :literal:`range`.


Track mtable
------------
.. _sec.track.mtable:

The :var:`track` command returns a *mtable* where the information described hereafter is the default list of fields written to the TFS files. [#f4]_

The header of the *mtable* contains the fields in the default order:

**name**
    The name of the command that created the *mtable*, e.g. :literal:`"track"`.
**type**
    The type of the *mtable*, i.e. :literal:`"track"`.
**origin**
    The origin of the application that created the *mtable*, e.g. :literal:`"MAD 1.0.0 OSX 64"`.
**date**
    The date of the creation of the *mtable*, e.g. :literal:`"27/05/20"`.
**time**
    The time of the creation of the *mtable*, e.g. :literal:`"19:18:36"`.
**refcol**
    The reference *column* for the *mtable* dictionnary, e.g. :literal:`"name"`.
**direction**
    The value of the command attribute :literal:`dir`.
**observe**
    The value of the command attribute :literal:`observe`.
**implicit**
    The value of the command attribute :literal:`implicit`.
**misalign**
    The value of the command attribute :literal:`misalign`.
**radiate**
    The value of the command attribute :literal:`radiate`.
**particle**
    The reference beam particle name.
**energy**
    The reference beam energy.
**deltap**
    The value of the command attribute :literal:`deltap`.
**lost**
    The number of lost particle(s) or damap(s).

The core of the *mtable* contains the columns in the default order:

**name**
    The name of the element.
**kind**
    The kind of the element.
**s**
    The :math:`s`-position at the end of the element slice.
**l**
    The length from the start of the element to the end of the element slice.
**id**
    The index of the particle or damap as provided in :var:`X0`.
**x**
    The local coordinate :math:`x` at the :math:`s`-position.
**px**
    The local coordinate :math:`p_x` at the :math:`s`-position.
**y**
    The local coordinate :math:`y` at the :math:`s`-position.
**py**
    The local coordinate :math:`p_y` at the :math:`s`-position.
**t**
    The local coordinate :math:`t` at the :math:`s`-position.
**pt**
    The local coordinate :math:`p_t` at the :math:`s`-position.
**pc**
    The reference beam :math:`P_0c` in which :math:`p_t` is expressed.
**ktap**
    The tapering compensation factor stored at the current tracking position.
**slc**
    The slice index ranging from :literal:`-2` to :literal:`nslice`.
**turn**
    The turn number.
**tdir**
    The :math:`t`-direction of the tracking in the element.
**eidx**
    The index of the element in the sequence.
**status**
    The status of the particle or damap.
**__map**
    The damap at the :math:`s`-position. [#f6]_

The mtable also carries the in-memory attributes :literal:`taper`, :literal:`range` and the protected sequence handle :literal:`__seq`, but they are not part of the default header list written to TFS files.


Dynamical tracking
------------------

:numref:`fig track trkslc` presents the scheme of the dynamical tracking through an element sliced with :literal:`nslice=3`. The actions :literal:`atentry` (index :literal:`-1`), :literal:`atslice` (indexes :literal:`0..3`), and :literal:`atexit` (index :literal:`-2`) are reversed between the forward tracking (:literal:`dir=1` with increasing :math:`s`-position) and the backward tracking (:literal:`dir=-1` with decreasing :math:`s`-position). By default, the action :literal:`atsave` is attached to the exit slice and the action :literal:`ataper` is attached to the last slice just before exit, i.e. to the last :literal:`atslice` action in the tilted frame, and hence they are also both reversed in the backward tracking.

.. _fig track trkslc:

.. figure:: fig/dyna-trck-slice-crop.png
    :align: center
    :figwidth: 98%

    Dynamical tracking with slices.

Slicing
"""""""

The slicing can take three different forms:

    *	 A *number* of the form :expr:`nslice=N` that specifies the number of slices with indexes :math:`0`..:math:`N`. This defines a uniform slicing with slice length :math:`l_{\text{slice}} = l_{\text{elem}}/N`.

    *	 An *iterable* of the form :literal:`nslice={lw_1,lw_2,..,lw_N}` with :math:`\sum_i lw_i=1` that specifies the fraction of length of each slice with indexes :math:`0` .. :math:`N` where :math:`N`\ =\ :literal:`#nslice`. This defines a non-uniform slicing with a slice length of :math:`l_i = lw_i\times l_{\text{elem}}`.

    *	 A *callable* :literal:`(elm, mflw, lw)` returning one of the two previous forms of slicing. The arguments are in order, the current element, the tracked map flow, and the length weight of the step, which should allow to return a user-defined element-specific slicing.


The surrounding :math:`P` and :math:`P^{-1}` maps represent the patches applied around the body of the element to change the frames, after the :literal:`atentry` and before the :literal:`atexit` actions:

    *	 The misalignment of the element to move from the *global frame* to the *element frame* if the command attribute :literal:`misalign` is set to :const:`true`.

    *	 The tilt of the element to move from the element frame to the *titled frame* if the element attribute :literal:`tilt` is non-zero. The :literal:`atslice` actions take place in this frame.

The *map frame* is specific to some maps while tracking through the body of the element. In principle, the map frame is not visible to the user, only to the integrator. For example, a quadrupole with both :literal:`k1` and :literal:`k1s` defined will have a *map frame* tilted by the angle :math:`\alpha=-\frac{1}{2}\tan^{-1}\frac{k1s}{k1}` attached to its thick map, i.e. the focusing matrix handling only :math:`\tilde{k}_1 = \sqrt{k1^2+k1s^2}`, but not to its thin map, i.e. the kick from all multipoles (minus :literal:`k1` and :literal:`k1s`) expressed in the *tilted frame*\ , during the integration steps.

Sub-elements
""""""""""""

The :var:`track` command takes sub-elements into account. In this case, the slicing specification is taken between sub-elements, e.g. 3 slices with 2 sub-elements gives a final count of 9 slices. It is possible to adjust the number of slices between sub-elements with the third form of slicing specifier, i.e. by using a callable where the length weight argument is between the current (or the end of the element) and the last sub-elements (or the start of the element).

Particles status
""""""""""""""""

The :var:`track` command initializes the map flow with particles or damaps or both, depending on the attributes :var:`X0` and :literal:`mapdef`. The :literal:`status` attribute of each particle or damap will be set to one of :literal:`"Xset"`, :literal:`"Mset"`, and :literal:`"Aset"` to track the origin of its initialization: coordinates, damap, or normalizing damap (normal form or beta block). After the tracking, some particles or damaps may have the status :literal:`"lost"` and their number being recorded in the counter :literal:`lost` from TFS table header. Other commands like :var:`cofind` or :var:`twiss` may add extra tags to the status value, like :literal:`"stable"`, :literal:`"unstable"` and :literal:`"singular"`.

Examples
--------



.. rubric:: Footnotes

.. [#f1] MAD-NG implements only two tracking codes denominated the *geometric* and the *dynamic* tracking.
.. [#f2] Initial coordinates :var:`X0` may override it by providing per particle or damap beam.
.. [#f3] The :literal:`TKT` scheme (Yoshida) is automatically converted to the :literal:`MKM` scheme (Boole) when approriate.
.. [#f7] In all cases, MAD-NG uses PTC setup :literal:`time=true, exact=true`.
.. [#f4] The output of mtable in TFS files can be fully customized by the user.
.. [#f6] Fields and columns starting with two underscores are protected data and never saved to TFS files.
