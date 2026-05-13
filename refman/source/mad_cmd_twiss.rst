Twiss
=====
.. _ch.cmd.twiss:

The :var:`twiss` command provides a simple interface to compute the optical functions around an orbit on top of the :var:`track` command, and the :var:`cofind` command if the search for closed orbits is requested.

Command synopsis
----------------
.. _sec.twiss.synop:

The :var:`twiss` command format is summarized in :numref:`fig-twiss-synop`, including the default setup of the attributes. Most of these attributes are set to :const:`nil` by default, meaning that :var:`twiss` relies on the :var:`track` and the :var:`cofind` commands defaults.

.. code-block:: lua
	:name: fig-twiss-synop
	:caption: Synopsis of the :var:`twiss` command with default setup.


	mtbl, mflw [, eidx] = twiss {
		sequence=sequ,  -- sequence (required)
		beam=nil, 	-- beam (or sequence.beam, required)
		range=nil,  	-- range of tracking (or sequence.range)
		dir=nil,  	-- s-direction of tracking (1 or -1)
		s0=nil,  	-- initial s-position offset [m]
		X0=nil,  	-- initial coordinates (or damap(s), or beta block(s))
		O0=nil,  	-- initial coordinates of reference orbit
		deltap=nil,  	-- initial deltap(s)
		chrom=false,  	-- compute chromatic functions by finite difference
		coupling=false, -- compute optical functions for non-diagonal modes
		trkopt=false,  	-- compute selected parametric optical functions
		trkrdt=false,  	-- compute selected resonance driving terms
		nturn=nil,  	-- number of turns to track
		nstep=nil,  	-- number of elements to track
		nslice=nil,  	-- number of slices (or weights) for each element
		method=nil,  	-- method or order for integration (1 to 8)
		model=nil,  	-- model for integration ('DKD' or 'TKT')
		mapdef=2,  	-- setup for damap (or list of, true => {xy=1})
		secnmul=nil,  	-- curved-multipole expansion for bends
		implicit=nil,  	-- slice implicit elements too (e.g. plots)
		misalign=nil,  	-- consider misalignment
		aperture=nil,  	-- default aperture
		fringe=nil,  	-- enable fringe fields (see element.flags.fringe)
		frngmax=nil,  	-- maximum multipole fringe field order
		radiate=nil,  	-- radiate at slices
		nocavity=nil,  	-- disable rfcavities
		totalpath=nil,  -- variable 't' is the totalpath
		cmap=nil,  	-- use C/C++ maps when available
		ptcmodel=nil,  	-- nil=MADNG, true=PTC, false=MADX
		save=true,  	-- create mtable and save results
		aper=nil,  	-- check for aperture
		observe=0,  	-- save only in observed elements (every n turns)
		savesel=nil,  	-- save selector (predicate)
		savemap=nil,  	-- save damap in the column __map
		saveanf=nil,  	-- save analysed normal form in the column __nf
		atentry=nil,  	-- action called when entering an element
		atslice=nil,  	-- action called after each element slices
		atexit=nil,  	-- action called when exiting an element
		ataper=nil,  	-- action called when checking for aperture
		atsave=nil,  	-- action called when saving in mtable
		atdebug=nil,  	-- action called when debugging the element maps
		apersel=nil,  	-- aperture selector (predicate)
		costp=nil,  	-- finite-difference scale for Jacobian
		coitr=nil,  	-- maximum number of iterations
		cotol=nil,  	-- closed orbit tolerance (i.e.|dX|)
		O1=nil,  	-- optional final coordinates translation
		info=nil,  	-- information level (output on terminal)
		debug=nil, 	-- debug information level (output on terminal)
		usrdef=nil,  	-- user defined data attached to the mflow
		mflow=nil,  	-- mflow, exclusive with other attributes
	}

The :var:`twiss` command supports the following attributes:

.. _twiss.attr:

**sequence**
	The *sequence* to track. (no default, required).

	Example: :expr:`sequence = lhcb1`.

**beam**
	The reference *beam* for the tracking. If no beam is provided, the command looks for a beam attached to the sequence, i.e. the attribute :literal:`seq.beam` . [#f1]_ (default: :const:`nil`).

	Example: :expr:`beam = beam 'lhcbeam' { beam-attributes }`.

**range**
	A *range* specifying the span of the sequence track. If no range is provided, the command looks for a range attached to the sequence, i.e. the attribute :attr:`seq.range`. (default: :const:`nil`).

	Example: :expr:`range = "S.DS.L8.B1/E.DS.R8.B1"`.

**dir**
	The :math:`s`-direction of the tracking: :const:`1` forward, :const:`-1` backward. (default: :const:`nil`).

	Example: :expr:`dir = -1`.

**s0**
	A *number* specifying the initial :math:`s`-position offset. (default: :const:`nil`).

	Example: :expr:`s0 = 5000`.

**X0**
	A *mappable* (or a list of *mappable*) specifying initial coordinates :literal:`{x,px,y,py, t,pt}`, damap, or beta0 block for each tracked object, i.e. particle or damap. The beta0 blocks are converted to damaps, while the coordinates are converted to damaps only if :literal:`mapdef` is specified, but both will use :literal:`mapdef` to setup the damap constructor. A closed orbit will be automatically searched for damaps built from coordinates. Each tracked object may also contain a :var:`beam` to override the reference beam, and a *logical* :literal:`nosave` to discard this object from being saved in the mtable. (default: :const:`nil`).

	Example: :expr:`X0 = { x=1e-3, px=-1e-5 }`.

**O0**
	A *mappable* specifying initial coordinates :literal:`{x,px,y,py,t,pt}` of the reference orbit around which X0 definitions take place. If it has the attribute :literal:`cofind == true`, it will be used as an initial guess to search for the reference closed orbit. (default: :const:`nil`).

	Example: :expr:`O0 = { x=1e-4, px=-2e-5, y=-2e-4, py=1e-5 }`.

**deltap**
	A *number* (or list of *number*) specifying the initial :math:`\delta_p` to convert (using the beam) and add to the :literal:`pt` of each tracked particle or damap. (default: :const:`nil`).

	Example: :expr:`s0 = 5000`.

**chrom**
	A *logical* specifying to calculate the chromatic functions by finite different using an extra :math:`\delta_p=` :const:`1e-6`. (default: :const:`false`).

	Example: :expr:`chrom = true`.

**coupling**
	A *logical* specifying to calculate the optical functions for coupling terms in the normalized forms. (default: :const:`false`).

	Example: :expr:`coupling = true`.

**trkopt**
	A *list* of optical-function names requesting additional parametric optics columns to be tracked and stored in the output table. (default: :const:`false`).

	Example: :expr:`trkopt = {'beta11_01', 'beta22_10'}`.

**trkrdt**
	A *list* of resonance-driving-term names requesting additional RDT columns to be tracked and stored in the output table. Each requested name must start with :literal:`'f'`. (default: :const:`false`).

	Example: :expr:`trkrdt = {'f3000', 'f2100'}`.

**nturn**
	A *number* specifying the number of turn to track. (default: :const:`nil`).

	Example: :expr:`nturn = 2`.

**nstep**
	A *number* specifying the number of element to track. A negative value will track all elements. (default: :const:`nil`).

	Example: :expr:`nstep = 1`.

**nslice**
	A *number* specifying the number of slices or an *iterable* of increasing relative positions or a *callable* :literal:`(elm, mflw, lw)` returning one of the two previous kind of positions to track in the elements. The arguments of the callable are in order, the current element, the tracked map flow, and the length weight of the step. This attribute can be locally overridden by the element. (default: :const:`nil`).

	Example: :expr:`nslice = 5`.

**mapdef**
	A *logical*, a *number*, or a *damap* specification as defined by the :doc:`DAmap <mad_mod_diffmap>` module to track DA maps instead of particles coordinates. A value of :const:`true` is equivalent to invoke the *damap* constructor with :literal:`{}` as argument. The default value :literal:`2` forces a second-order damap setup for the linear normal-form analysis. (default: :literal:`2`).

	Example: :expr:`mapdef = { xy=2, pt=5 }`.

**method**
	A *number* specifying the order of integration from 1 to 8, or a *string* specifying a special method of integration. Odd orders are rounded to the next even order to select the corresponding Yoshida or Boole integration schemes. The special methods are :literal:`simple` (equiv. to :literal:`DKD` order 2), :literal:`collim` (equiv. to :literal:`MKM` order 2), and :literal:`teapot` (Teapot splitting order 2). (default: :const:`nil`).

	Example: :expr:`method = 'teapot'`.

**model**
	A *string* specifying the integration model, either :literal:`'DKD'` for *Drift-Kick-Drift* thin lens integration or :literal:`'TKT'` for *Thick-Kick-Thick* thick lens integration. [#f7]_ (default: :const:`nil`)

	Example: :expr:`model = 'DKD'`.

**ptcmodel**
	A *logical* selecting the tracking backend: :const:`nil` keeps the MAD-NG model, :const:`true` selects the strict PTC model, and :const:`false` selects the MAD-X-compatible model. [#f8]_ (default: :const:`nil`)

	Example: :expr:`ptcmodel = true`.

The attributes :literal:`secnmul`, :literal:`aperture`, :literal:`frngmax`, :literal:`nocavity`, :literal:`cmap`, :literal:`aper` and :literal:`apersel` are forwarded unchanged to the underlying :var:`track` command and therefore keep the same semantics and default handling as in :doc:`track <mad_cmd_track>`.

**implicit**
	A *logical* indicating that implicit elements must be sliced too, e.g. for smooth plotting. (default: :const:`nil`).

	Example: :expr:`implicit = true`.

**misalign**
	A *logical* indicating that misalignment must be considered. (default: :const:`nil`).

	Example: :expr:`misalign = true`.

**fringe**
	A *logical* indicating that fringe fields must be considered or a *number* specifying a bit mask to apply to all elements fringe flags defined by the element module. The value :const:`true` is equivalent to the bit mask , i.e. allow all elements (default) fringe fields. (default: :const:`nil`).

	Example: :expr:`fringe = false`.

**radiate**
	A *logical* enabling damping radiation during the closed-orbit and one-turn-map preparation phases. Any truthy value is collapsed to the damping-only tracking mode used internally by :var:`twiss`. (default: :const:`nil`).

	Example: :expr:`radiate = true`.

**totalpath**
	A *logical* indicating to use the totalpath for the fifth variable :literal:`'t'` instead of the local path. (default: :const:`nil`).

	Example: :expr:`totalpath = true`.

**save**
	A *logical* specifying to create a *mtable* and record tracking information at the observation points. The :literal:`save` attribute can also be a *string* specifying saving positions in the observed elements: :literal:`"atentry"`, :literal:`"atslice"`, :literal:`"atexit"` (i.e. :const:`true`), :literal:`"atbound"` (i.e. entry and exit), :literal:`"atbody"` (i.e. slices and exit) and :literal:`"atall"`. (default: :const:`true`).

	Example: :expr:`save = false`.

**observe**
	A *number* specifying the observation points to consider for recording the tracking information. A zero value will consider all elements, while a positive value will consider selected elements only, checked with method :meth:`:is_observed`, every :literal:`observe`\ :math:`>0` turns. (default: :const:`0`).

	Example: :expr:`observe = 1`.

**savesel**
	A *callable* :literal:`(elm, mflw, lw, islc)` acting as a predicate on selected elements for observation, i.e. the element is discarded if the predicate returns :const:`false`. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :const:`fnil`)

	Example: :expr:`savesel = \\e -> mylist[e.name] ~= nil`.

**savemap**
	A *logical* indicating to save the damap in the column :literal:`__map` of the *mtable*. (default: :const:`nil`).

	Example: :expr:`savemap = true`.

**saveanf**
	A *logical* indicating to save the analysed normal form in the protected column :literal:`__nf` of the *mtable*. (default: :const:`nil`).

	Example: :expr:`saveanf = true`.

**atentry**
	A *callable* :literal:`(elm, mflw, 0, -1)` invoked at element entry. The arguments are in order, the current element, the tracked map flow, zero length and the slice index :const:`-1`. (default: :const:`fnil`).

	Example: :expr:`atentry = myaction`.

**atslice**
	A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element slice. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :const:`fnil`).

	Example: :expr:`atslice = myaction`.

**atexit**
	A *callable* :literal:`(elm, mflw, 0, -2)` invoked at element exit. The arguments are in order, the current element, the tracked map flow, zero length and the slice index . (default: :const:`fnil`).

	Example: :expr:`atexit = myaction`.

**ataper**
	A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element aperture checks, by default at last slice. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. If a particle or a damap hits the aperture, then its :literal:`status="lost"` and it is removed from the list of tracked items. (default: :const:`fnil`).

	Example: :expr:`ataper = myaction`.

**atsave**
	A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element saving steps, by default at exit. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :const:`fnil`).

	Example: :expr:`atsave = myaction`.

**atdebug**
	A *callable* :literal:`(elm, mflw, lw, [msg], [...])` invoked at the entry and exit of element maps during the integration steps, i.e. within the slices. The arguments are in order, the current element, the tracked map flow, the length weight of the integration step and a *string* specifying a debugging message, e.g. :literal:`"map_name:0"` for entry and :literal:`":1"` for exit. If the level :literal:`debug` :math:`\geq 4` and :literal:`atdebug` is not specified, the default *function* :literal:`mdump` is used. In some cases, extra arguments could be passed to the method. (default: :const:`fnil`).

	Example: :expr:`atdebug = myaction`.

**costp**
	A *number* specifying the relative finite-difference scale to approximate the Jacobian when damaps are disabled. If :literal:`costp == true`, the closed-orbit preparation stage falls back to the default finite-difference scale from :var:`cofind`. (default: :const:`nil`).

	Example: :expr:`costp = 1e-8`.

**coitr**
	A *number* specifying the maximum number of iteration for the closed-orbit search stage. If this threshold is reached, all the remaining tracked objects are tagged as :literal:`"unstable"`. (default: :const:`nil` and inherited from :var:`cofind`).

	Example: :expr:`coitr = 5`.

**cotol**
	A *number* specifying the closed orbit tolerance. If all coordinates update of a particle or a damap are smaller than :literal:`cotol`, then it is tagged as :literal:`"stable"`. (default: :const:`nil` and inherited from :var:`cofind`).

	Example: :expr:`cotol = 1e-6`.

**O1**
	A *mappable* specifying the coordinates :literal:`{x,px,y,py,t,pt}` to *subtract* from the final closed-orbit coordinates. (default: :const:`nil`).

	Example: :expr:`O1 = { t=100, pt=10 }`.

**info**
	A *number* specifying the information level to control the verbosity of the output on the console. (default: :const:`nil`).

	Example: :expr:`info = 2`.

**debug**
	A *number* specifying the debug level to perform extra assertions and to control the verbosity of the output on the console. (default: :const:`nil`).

	Example: :expr:`debug = 2`.

**usrdef**
	Any user defined data that will be attached to the tracked map flow, which is internally passed to the elements method :literal:`:track` and to their underlying maps. (default: :const:`nil`).

	Example: :expr:`usrdef = { myvar=somevalue }`.

**mflow**
	A *mflow* containing the current state of a :var:`twiss` command. If a map flow is provided, all setup attributes are ignored and only the volatile fields already stored in the mflow are reused; in practice, :literal:`nstep`, :literal:`info` and :literal:`debug` are the fields refreshed by the command wrapper. (default: :const:`nil`).

	Example: :expr:`mflow = mflow0`.


The :var:`twiss` command returns the following objects in this order:

**mtbl**
	A *mtable* corresponding to the augmented TFS table of the :var:`track` command with the :var:`twiss` command columns.

**mflw**
	A *mflow* corresponding to the augmented map flow of the :var:`track` command with the :var:`twiss` command data.

**eidx**
	An optional *number* corresponding to the last tracked element index in the sequence when :literal:`nstep` was specified and stopped the command before the end of the :literal:`range`.


Twiss mtable
------------
.. _sec.twiss.mtable:

The :var:`twiss` command returns a *mtable* where the information described hereafter is the default list of fields written to the TFS files. [#f2]_

The header of the *mtable* contains the fields in the default order: [#f3]_

**name**
	The name of the command that created the *mtable*, e.g. :literal:`"twiss"`.
**type**
	The type of the *mtable*, i.e. :literal:`"twiss"`.
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
**chrom**
	The value of the command attribute :literal:`chrom`.
**coupling**
	The value of the command attribute :literal:`coupling`.
**trkopt**
	The value of the command attribute :literal:`trkopt`.
**trkrdt**
	The value of the command attribute :literal:`trkrdt`.
**length**
	The :math:`s`-length of the tracked design orbit.
**q1**
	The tunes of mode 1.
**q2**
	The tunes of mode 2.
**q3**
	The tunes of mode 3.
**alfap**
	The momentum compaction factor :math:`\alpha_p`.
**etap**
	The phase slip factor :math:`\eta_p`.
**gammatr**
	The energy gamma transition :math:`\gamma_{\text{tr}}`.
**synch_1**
	The first synchroton radiation integral.
**synch_2**
	The second synchroton radiation integral.
**synch_3**
	The third synchroton radiation integral.
**synch_4**
	The fourth synchroton radiation integral.
**synch_5**
	The fifth synchroton radiation integral.
**synch_6**
	The sixth synchroton radiation integral.
**synch_8**
	The eighth synchroton radiation integral.

The core of the *mtable* contains the columns in the default order: [#f6]_

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
	The local coordinate :math:`x` at the :math:`s`-position .
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
**alfa11**
	The optical function :math:`\alpha` of mode 1 at the :math:`s`-position.
**beta11**
	The optical function :math:`\beta` of mode 1 at the :math:`s`-position.
**gama11**
	The optical function :math:`\gamma` of mode 1 at the :math:`s`-position.
**mu1**
	The phase advance :math:`\mu` of mode 1 at the :math:`s`-position.
**dx**
	The dispersion function of :math:`x` at the :math:`s`-position.
**dpx**
	The dispersion function of :math:`p_x` at the :math:`s`-position.
**alfa22**
	The optical function :math:`\alpha` of mode 2 at the :math:`s`-position.
**beta22**
	The optical function :math:`\beta` of mode 2 at the :math:`s`-position.
**gama22**
	The optical function :math:`\gamma` of mode 2 at the :math:`s`-position.
**mu2**
	The phase advance :math:`\mu` of mode 2 at the :math:`s`-position.
**dy**
	The dispersion function of :math:`y` at the :math:`s`-position.
**dpy**
	The dispersion function of :math:`p_y` at the :math:`s`-position.
**alfa33**
	The optical function :math:`\alpha` of mode 3 at the :math:`s`-position.
**beta33**
	The optical function :math:`\beta` of mode 3 at the :math:`s`-position.
**gama33**
	The optical function :math:`\gamma` of mode 3 at the :math:`s`-position.
**mu3**
	The phase advance :math:`\mu` of mode 3 at the :math:`s`-position.
**__map**
	The damap at the :math:`s`-position. [#f5]_

If :literal:`saveanf == true`, the protected column :literal:`__nf` is also added and stores the analysed normal form at each saved row.

The mtable also carries the in-memory attributes :literal:`taper`, :literal:`range` and the protected sequence handle :literal:`__seq`, but they are not part of the default header list written to TFS files.

The :literal:`chrom` attribute will add the following fields to the *mtable* header:

**dq1**
	The chromatic derivative of tunes of mode 1, i.e. chromaticities.
**dq2**
	The chromatic derivative of tunes of mode 2, i.e. chromaticities.
**dq3**
	The chromatic derivative of tunes of mode 3, i.e. chromaticities.

The :literal:`chrom` attribute will add the following columns to the *mtable*:

**dmu1**
	The chromatic derivative of the phase advance of mode 1 at the :math:`s`-position.
**ddx**
	The chromatic derivative of the dispersion function of :math:`x` at the :math:`s`-position.
**ddpx**
	The chromatic derivative of the dispersion function of :math:`p_x` at the :math:`s`-position.
**wx**
	The chromatic amplitude function of mode 1 at the :math:`s`-position.
**phix**
	The chromatic phase function of mode 1 at the :math:`s`-position.
**dmu2**
	The chromatic derivative of the phase advance of mode 2 at the :math:`s`-position.
**ddy**
	The chromatic derivative of the dispersion function of :math:`y` at the :math:`s`-position.
**ddpy**
	The chromatic derivative of the dispersion function of :math:`p_y` at the :math:`s`-position.
**wy**
	The chromatic amplitude function of mode 2 at the :math:`s`-position.
**phiy**
	The chromatic phase function of mode 2 at the :math:`s`-position.

The :literal:`coupling` attribute will add the following columns to the *mtable*:

**alfa12**
	The optical function :math:`\alpha` of coupling mode 1-2 at the :math:`s`-position.
**beta12**
	The optical function :math:`\beta` of coupling mode 1-2 at the :math:`s`-position.
**gama12**
	The optical function :math:`\gamma` of coupling mode 1-2 at the :math:`s`-position.
**alfa13**
	The optical function :math:`\alpha` of coupling mode 1-3 at the :math:`s`-position.
**beta13**
	The optical function :math:`\beta` of coupling mode 1-3 at the :math:`s`-position.
**gama13**
	The optical function :math:`\gamma` of coupling mode 1-3 at the :math:`s`-position.
**alfa21**
	The optical function :math:`\alpha` of coupling mode 2-1 at the :math:`s`-position.
**beta21**
	The optical function :math:`\beta` of coupling mode 2-1 at the :math:`s`-position.
**gama21**
	The optical function :math:`\gamma` of coupling mode 2-1 at the :math:`s`-position.
**alfa23**
	The optical function :math:`\alpha` of coupling mode 2-3 at the :math:`s`-position.
**beta23**
	The optical function :math:`\beta` of coupling mode 2-3 at the :math:`s`-position.
**gama23**
	The optical function :math:`\gamma` of coupling mode 2-3 at the :math:`s`-position.
**alfa31**
	The optical function :math:`\alpha` of coupling mode 3-1 at the :math:`s`-position.
**beta31**
	The optical function :math:`\beta` of coupling mode 3-1 at the :math:`s`-position.
**gama31**
	The optical function :math:`\gamma` of coupling mode 3-1 at the :math:`s`-position.
**alfa32**
	The optical function :math:`\alpha` of coupling mode 3-2 at the :math:`s`-position.
**beta32**
	The optical function :math:`\beta` of coupling mode 3-2 at the :math:`s`-position.
**gama32**
	The optical function :math:`\gamma` of coupling mode 3-2 at the :math:`s`-position.


Tracking linear normal form
---------------------------

TODO

Examples
--------

TODO


.. rubric:: Footnotes

.. [#f1] Initial coordinates :var:`X0` may override it by providing a beam per particle or damap.
.. [#f7] The :literal:`TKT` scheme (Yoshida) is automatically converted to the :literal:`MKM` scheme (Boole) when appropriate.
.. [#f8] In all cases, MAD-NG uses PTC setup :literal:`time=true, exact=true`.
.. [#f2] The output of mtable in TFS files can be fully customized by the user.
.. [#f3] The fields from :literal:`name` to :literal:`lost` are set by the :var:`track` command
.. [#f5] Fields and columns starting with two underscores are protected data and never saved to TFS files.
.. [#f6] The column from :literal:`name` to :literal:`status` are set by the :var:`track` command.
