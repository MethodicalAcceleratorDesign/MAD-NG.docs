Survey
======
.. _ch.cmd.survey:

The :literal:`survey` command provides a simple interface to the *geometric* tracking code. [#f1]_ The geometric tracking can be used to place the elements of a sequence in the global reference system in :numref:`fig-phy-grs`.

.. _fig.survey.synop:

.. code-block:: lua
	:caption: Synopsis of the :literal:`survey` command with default setup.

	mtbl, mflw [, eidx] = survey { 
		sequence=sequ,  -- sequence (required) 
		range=nil,  	-- range of tracking (or sequence.range) 
		dir=1,  	-- s-direction of tracking (1 or -1) 
		s0=0,  		-- initial s-position offset [m] 
		X0=0,  		-- initial coordinates x, y, z [m] 
		A0=0,  		-- initial angles theta, phi, psi [rad] or matrix W0 
		nturn=1,  	-- number of turns to track 
		nstep=-1,  	-- number of elements to track 
		nslice=1,  	-- number of slices (or weights) for each element 
		implicit=false,  -- slice implicit elements too (e.g.~plots) 
		misalign=false,  -- consider misalignment 
		save=true,  	-- create mtable and save results 
		title=nil,  	-- title of mtable (default seq.name) 
		observe=0,  	-- save only in observed elements (every n turns) 
		savesel=fnil,  	-- save selector (predicate) 
		savemap=false,  -- save the orientation matrix W in the column __map 
		atentry=fnil,  	-- action called when entering an element 
		atslice=fnil,  	-- action called after each element slices 
		atexit=fnil,  	-- action called when exiting an element 
		atsave=fnil,  	-- action called when saving in mtable 
		atdebug=fnil,  	-- action called when debugging the element maps 
		info=nil,  	-- information level (output on terminal) 
		debug=nil, 	-- debug information level (output on terminal) 
		usrdef=nil,  	-- user defined data attached to the mflow 
		mflow=nil,  	-- mflow, exclusive with other attributes except nstep 
	}

Command synopsis
----------------
.. _sec.survey.synop:


The :literal:`survey` command format is summarized in :numref:`fig.survey.synop`, including the default setup of the attributes. The :literal:`survey` command supports the following attributes:

.. _survey.attr:

**sequence**
	 The *sequence* to survey. (no default, required). 
 	 Example: :expr:`sequence = lhcb1`.

**range** 
	 A *range* specifying the span of the sequence survey. If no range is provided, the command looks for a range attached to the sequence, i.e. the attribute . (default: :const:`nil`). 
	 
	 Example: :expr:`range = "S.DS.L8.B1/E.DS.R8.B1"`.

**dir** 
	 The :math:`s`-direction of the tracking: :const:`1` forward, :const:`-1` backward. (default: :const:`1`). 

	 Example: :expr:`dir = -1`.

**s0** 
	 A *number* specifying the initial :math:`s`-position offset. (default: :const:`0` [m]). 

	 Example: :expr:`s0 = 5000`.

**X0** 
	 A *mappable* specifying the initial coordinates :literal:`{x,y,z}`. (default: :const:`0` [m]). 

	 Example: :expr:`X0 = { x=100, y=-50 }`

**A0** 
	 A *mappable* specifying the initial angles :literal:`theta`, :literal:`phi` and :literal:`psi` or an orientation *matrix* :literal:`W0`. [#f2]_ (default: :const:`0` [rad]). 

	 Example: :expr:`A0 = { theta=deg2rad(30) }`

**nturn** 
	 A *number* specifying the number of turn to track. (default: :literal:`1`). 

	 Example: :expr:`nturn = 2`.

**nstep** 
	 A *number* specifying the number of element to track. A negative value will track all elements. (default: :const:`-1`). 

	 Example: :expr:`nstep = 1`.

**nslice** 
	 A *number* specifying the number of slices or an *iterable* of increasing relative positions or a *callable* :literal:`(elm, mflw, lw)` returning one of the two previous kind of positions to track in the elements. The arguments of the callable are in order, the current element, the tracked map flow, and the length weight of the step. This attribute can be locally overridden by the element. (default: :const:`1`). 

	 Example: :expr:`nslice = 5`.

**implicit** 
	 A *logical* indicating that implicit elements must be sliced too, e.g. for smooth plotting. (default: :const:`false`). 

	 Example: :expr:`implicit = true`.

**misalign** 
	 A *logical* indicating that misalignment must be considered. (default: :const:`false`). 

	 Example: :expr:`implicit = true`.

**save** 
	 A *logical* specifying to create a *mtable* and record tracking information at the observation points. The :literal:`save` attribute can also be a *string* specifying saving positions in the observed elements: :literal:`"atentry"`, :literal:`"atslice"`, :literal:`"atexit"` (i.e. :const:`true`), :literal:`"atbound"` (i.e. entry and exit), :literal:`"atbody"` (i.e. slices and exit) and :literal:`"atall"`. (default: :const:`true`). 

	 Example: :expr:`save = false`.

**title** 
	 A *string* specifying the title of the *mtable*. If no title is provided, the command looks for the name of the sequence, i.e. the attribute :literal:`seq.name`. (default: :const:`nil`). 

	 Example: :expr:`title = "Survey around IP5"`.

**observe** 
	 A *number* specifying the observation points to consider for recording the tracking information. A zero value will consider all elements, while a positive value will consider selected elements only, checked with method :literal:`:is_observed`, every :literal:`observe` :math:`>0` turns. (default: :const:`0`). 

	 Example: :expr:`observe = 1`.

**savesel** 
	 A *callable* :literal:`(elm, mflw, lw, islc)` acting as a predicate on selected elements for observation, i.e. the element is discarded if the predicate returns :const:`false`. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`) 
	 Example: :expr:`savesel = \\e -> mylist[e.name] ~= nil`.

**savemap** 
	 A *logical* indicating to save the orientation matrix :literal:`W` in the column :literal:`__map` of the *mtable*. (default: :const:`false`). 

	 Example: :expr:`savemap = true`.

**atentry** 
	 A *callable* :literal:`(elm, mflw, 0, -1)` invoked at element entry. The arguments are in order, the current element, the tracked map flow, zero length and the slice index :const:`-1`. (default: :literal:`fnil`). 

	 Example: :expr:`atentry = myaction`.

**atslice** 
	 A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element slice. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`). 

	 Example: :expr:`atslice = myaction`.

**atexit** 
	 A *callable* :literal:`(elm, mflw, 0, -2)` invoked at element exit. The arguments are in order, the current element, the tracked map flow, zero length and the slice index :const:`-2`. (default: :literal:`fnil`). 

	 Example: :expr:`atexit = myaction`.

**atsave** 
	 A *callable* :literal:`(elm, mflw, lw, islc)` invoked at element saving steps, by default at exit. The arguments are in order, the current element, the tracked map flow, the length weight of the slice and the slice index. (default: :literal:`fnil`). 

	 Example: :expr:`atsave = myaction`.

**atdebug** 
	 A *callable* :literal:`(elm, mflw, lw, [msg], [...])` invoked at the entry and exit of element maps during the integration steps, i.e. within the slices. The arguments are in order, the current element, the tracked map flow, the length weight of the integration step and a *string* specifying a debugging message, e.g. :literal:`"map_name:0"` for entry and :literal:`":1"` for exit. If the level :literal:`debug` :math:`\geq 4` and :literal:`atdebug` is not specified, the default *function* :literal:`mdump` is used. In some cases, extra arguments could be passed to the method. (default: :literal:`fnil`). 

	 Example: :expr:`atdebug = myaction` .
	 
**info**
	 A *number* specifying the information level to control the verbosity of the output on the console. (default: :const:`nil`). 

	 Example: :expr:`info = 2`.

**debug**
	 A *number* specifying the debug level to perform extra assertions and to control the verbosity of the output on the console. (default: :const:`nil`). 

	 Example: :expr:`debug = 2`.

**usrdef** 
	 Any user defined data that will be attached to the tracked map flow, which is internally passed to the elements method :meth:`:survey` and to their underlying maps. (default: :const:`nil`). 

	 Example: :expr:`usrdef = { myvar=somevalue }`.

**mflow** 
	 A *mflow* containing the current state of a :literal:`survey` command. If a map flow is provided, all attributes are discarded except :literal:`nstep`, :literal:`info` and :literal:`debug`, as the command was already set up upon its creation. (default: :const:`nil`). 

	 Example: :expr:`mflow = mflow0`.


The :literal:`survey` command returns the following objects in this order:

**mtbl** 
	A *mtable* corresponding to the TFS table of the :literal:`survey` command.

**mflw** 
	A *mflow* corresponding to the map flow of the :literal:`survey` command.

**eidx**
	 An optional *number* corresponding to the last surveyed element index in the sequence when :literal:`nstep` was specified and stopped the command before the end of the :literal:`range`.


Survey mtable
-------------
.. _sec.survey.mtable:

The :literal:`survey` command returns a *mtable* where the information described hereafter is the default list of fields written to the TFS files. [#f3]_ 

The header of the *mtable* contains the fields in the default order:

**name**
	 The name of the command that created the *mtable*, e.g. :literal:`"survey"`.
**type**
	 The type of the *mtable*, i.e. :literal:`"survey"`.
**title**
	 The value of the command attribute :literal:`title`.
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
**range**
	 The value of the command attribute :literal:`range`. [#f4]_ 
**__seq**
	 The *sequence* from the command attribute :var:`sequence`. [#f5]_

The core of the *mtable* contains the columns in the default order:

**name**
	 The name of the element.
**kind**
	 The kind of the element.
**s**
	 The :math:`s`-position at the end of the element slice.
**l**
	 The length from the start of the element to the end of the element slice.
**angle**
	 The angle from the start of the element to the end of the element slice.
**tilt**
	 The tilt of the element.
**x**
	 The global coordinate :math:`x` at the :math:`s`-position.
**y**
	 The global coordinate :math:`y` at the :math:`s`-position.
**z**
	 The global coordinate :math:`z` at the :math:`s`-position.
**theta**
	 The global angle :math:`\theta` at the :math:`s`-position.
**phi**
	 The global angle :math:`\phi` at the :math:`s`-position.
**psi**
	 The global angle :math:`\psi` at the :math:`s`-position.
**slc**
	 The slice number ranging from :literal:`-2` to :literal:`nslice`.
**turn**
	 The turn number.
**tdir**
	 The :math:`t`-direction of the tracking in the element.
**eidx**
	 The index of the element in the sequence.
**__map**
	 The orientation *matrix* at the :math:`s`-position. [#f5]_


Geometrical tracking
--------------------

:numref:`fig.survey.trkslc` presents the scheme of the geometrical tracking through an element sliced with :literal:`nslice=3`. The actions :literal:`atentry` (index :literal:`-1`), :literal:`atslice` (indexes :literal:`0..3`), and :literal:`atexit` (index :literal:`-2`) are reversed between the forward tracking (:literal:`dir=1` with increasing :math:`s`-position) and the backward tracking (:literal:`dir=-1` with decreasing :math:`s`-position). By default, the action :literal:`atsave` is attached to the exit slice, and hence it is also reversed in the backward tracking.


.. _fig.survey.trkslc:
.. figure:: fig/dyna-trck-slice-crop.png

	Geometrical tracking with slices.

Slicing
"""""""

The slicing can take three different forms:

	*	 A *number* of the form :literal:`nslice=`:math:`N` that specifies the number of slices with indexes :literal:`0..N`. This defines a uniform slicing with slice length :math:`l_{\text{slice}} = l_{\text{elem}}/N`.

	*	 An *iterable* of the form :literal:`nslice={lw_1,lw_2,..lw_N}` with :math:`\sum_i lw_i=1` that specifies the fraction of length of each slice with indexes :literal:`0..N` where :math:`N=`\ :literal:`#nslice`. This defines a non-uniform slicing with a slice length of :math:`l_i = lw_i\times l_{\text{elem}}`.

	*	 A *callable* :literal:`(elm, mflw, lw)` returning one of the two previous forms of slicing. The arguments are in order, the current element, the tracked map flow, and the length weight of the step, which should allow to return a user-defined element-specific slicing. 


The surrounding :literal:`P` and :literal:`P`\ :math:`^{-1}` maps represent the patches applied around the body of the element to change the frames, after the :literal:`atentry` and before the :literal:`atexit` actions:

	*	 The misalignment of the element to move from the *global frame* to the *element frame* if the command attribute :literal:`misalign` is set to :const:`true`.

	*	 The tilt of the element to move from the element frame to the *titled frame* if the element attribute :literal:`tilt` is non-zero. The :literal:`atslice` actions take place in this frame.

These patches do not change the global frame per se, but they may affect the way that other components change the global frame, e.g. the tilt combined with the angle of a bending element.

Sub-elements
""""""""""""

The :literal:`survey` command takes sub-elements into account, mainly for compatibility with the :var:`track` command. In this case, the slicing specification is taken between sub-elements, e.g. 3 slices with 2 sub-elements gives a final count of 9 slices. It is possible to adjust the number of slices between sub-elements with the third form of slicing specifier, i.e. by using a callable where the length weight argument is between the current (or the end of the element) and the last sub-elements (or the start of the element).

Examples
--------



.. rubric:: Footnotes

.. [#f1] MAD-NG implements only two tracking codes denominated the *geometric* and *dynamic* tracking
.. [#f2] An orientation matrix can be obtained from the 3 angles with :literal:`W=matrix(3):rotzxy(- phi,theta,psi)`
.. [#f3] The output of mtable in TFS files can be fully customized by the user.
.. [#f4] This field is not saved in the TFS table by default.
.. [#f5] Fields and columns starting with two underscores are protected data and never saved to TFS files.
