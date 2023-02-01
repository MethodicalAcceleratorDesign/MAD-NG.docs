Correct
=======
.. _ch.cmd.correct:

The :literal:`correct` command (i.e. orbit correction) provides a simple interface to compute the orbit steering correction and setup the kickers of the sequences from the analysis of their :literal:`track` and :literal:`twiss` mtables.

.. code-block:: 
	:caption: Synopsis of the :literal:`correct` command with default setup.
	:name: fig-correct-synop

	mlst = correct { 
		sequence=nil,	-- sequence(s) (required) 
		range=nil,  	-- sequence(s) range(s) (or sequence.range) 
		title=nil,  	-- title of mtable (default seq.name) 
		model=nil,  	-- mtable(s) with twiss functions (required) 
		orbit=nil,  	-- mtable(s) with measured orbit(s), or use model 
		target=nil,  	-- mtable(s) with target orbit(s), or zero orbit 
		kind='ring',  	--\ 'line' or 'ring' 
		plane='xy',  	--\ 'x', 'y' or 'xy' 
		method='micado',--\ 'LSQ', 'SVD' or 'MICADO' 
		ncor=0,  	-- number of correctors to consider by method, 0=all 
		tol=1e-5,  	-- rms tolerance on the orbit 
		units=1,  	-- units in [m] of the orbit 
		corcnd=false,  	-- precond of correctors using 'svdcnd' or 'pcacnd' 
		corcut=0,  	-- value to theshold singular values in precond 
		cortol=0,  	-- value to theshold correctors in svdcnd 
		corset=true,  	-- update correctors correction strengths 
		monon=false,  	-- fraction (0<?<=1) of randomly available monitors 
		moncut=false,  	-- cut monitors above moncut sigmas 
		monerr=false,  	-- 1:use mrex and mrey alignment errors of monitors 
						-- 2:use msex and msey scaling errors of monitors 
		info=nil,  	-- information level (output on terminal) 
		debug=nil, 	-- debug information level (output on terminal) 
	}

.. _sec.correct.synop:

Command synopsis
----------------

The :literal:`correct` command format is summarized in :numref:`fig-correct-synop`, including the default setup of the attributes.
The :literal:`correct` command supports the following attributes:

.. _correct.attr:

	**sequence**
	 The *sequence* (or a list of *sequence*) to analyze. (no default, required). 
	 Example: :literal:`sequence = lhcb1`.

	**range**
	 A *range* (or a list of *range*) specifying the span of the sequence to analyze. If no range is provided, the command looks for a range attached to the sequence, i.e. the attribute . (default: :literal:`nil`). 
	 Example: :literal:`range = "S.DS.L8.B1/E.DS.R8.B1"`.

	**title**
	 A *string* specifying the title of the *mtable*. If no title is provided, the command looks for the name of the sequence, i.e. the attribute :literal:`seq.name`. (default: :literal:`nil`). 
	 Example: :literal:`title = "Correct orbit around IP5"`.

	**model** 
	 A *mtable* (or a list of *mtable*) providing :literal:`twiss`-like information, e.g. elements, orbits and optical functions, of the corresponding sequences. (no default, required). 
	 Example: :literal:`model = twmtbl`.

	**orbit**
	 A *mtable* (or a list of *mtable*) providing :literal:`track`-like information, e.g. elements and measured orbits, of the corresponding sequences. If this attribute is :literal:`nil`, the model orbit is used. (default: :literal:`nil`). 
	 Example: :literal:`orbit = tkmtbl`.

	**target** 
	 A *mtable* (or a list of *mtable*) providing :literal:`track`-like information, e.g. elements and target orbits, of the corresponding sequences. If this attribute is :literal:`nil`, the design orbit is used. (default: :literal:`nil`). 
	 Example: :literal:`target = tgmtbl`.

	**kind** 
	 A *string* specifying the kind of correction to apply among :literal:`line` or :literal:`ring`. The kind :literal:`line` takes care of the causality between monitors, correctors and sequences directions, while the kind :literal:`ring` considers the system as periodic. (default: ). 
	 Example: :literal:`kind = 'line'`.

	**plane**
	 A *string* specifying the plane to correct among :literal:`x`, , :literal:`y` and :literal:`xy`. (default: :literal:`'xy'`). 
	 Example: :literal:`plane = 'x'`.

	**method**
	 A *string* specifying the method to use for correcting the orbit among :literal:`LSQ`, :literal:`SVD` or :literal:`micado`. These methods correspond to the solver used from the :doc:`linear algebra <linalg>` module to find the orbit correction, namely :literal:`solve`, :literal:`ssolve` or :literal:`nsolve`. (default: :literal:`'micado'`). 
	 Example: :literal:`method = 'svd'`.

	**ncor**
	 A *number* specifying the number of correctors to consider with the method :literal:`micado`, zero meaning all available correctors. (default: :literal:`0`). 
	 Example: :literal:`ncor = 4`.

	**tol** 
	 A *number* specifying the rms tolerance on the residuals for the orbit correction. (default: 1e-6). 
	 Example: :literal:`tol = 1e- 6`.

	**unit**
	 A *number* specifying the unit of the :literal:`orbit` and :literal:`target` coordinates. (default: :literal:`1` [m]). 
	 Example: :literal:`units = 1e- 3` [m], i.e. [mm].

	**corcnd** 
	 A *log* or a *string* specifying the method to use among :literal:`svdcnd` and :literal:`pcacnd` from the :doc:`linear algebra <linalg>` module for the preconditioning of the system. A :literal:`true` value corresponds to . (default: :literal:`false`). 
	 Example: :literal:`corcnd = 'pcacnd'`.

	**corcut** 
	 A *number* specifying the thresholds for the singular values to pass to the :literal:`svdcnd` and :literal:`pcacnd` method for the preconditioning of the system. (default: :literal:`0`). 
	 Example: :literal:`cortol = 1e- 6`.

	**cortol**
	 A *number* specifying the thresholds for the correctors to pass to the :literal:`svdcnd` method for the preconditioning of the system. (default: :literal:`0`). 
	 Example: :literal:`cortol = 1e- 8`.

	**corset**
	 A *log* specifying to update the correctors strengths for the corrected orbit. (default: :literal:`true`). 
	 Example: :literal:`corset = false`.

	**monon**
	 A *number* specifying a fraction of available monitors selected from a uniform RNG. (default: :literal:`false`). 
	 Example: :literal:`monon = 0.8`, keep 80% of the monitors.

	**moncut**
	 A *number* specifying a threshold in number of sigma to cut monitor considered as outliers. (default: :literal:`false`). 
	 Example: :literal:`moncut = 2`, cut monitors above :math:`2\sigma`.

	**monerr**
	 A *number* in :literal:`0..3` specifying the type of monitor reading errors to consider: :literal:`1` use scaling errors :literal:`msex` and :literal:`msey`, :literal:`2` use alignment errors :literal:`mrex`, :literal:`mrey` and :literal:`dpsi`, :literal:`3` use both. (default: :literal:`false`). 
	 Example: :literal:`monerr = 3`.

	**info**
	 A *number* specifying the information level to control the verbosity of the output on the console. (default: :literal:`nil`). 
	 Example: :literal:`info = 2`.

	**debug**
	 A *number*\ specifying the debug level to perform extra assertions and to control the verbosity of the output on the console. (default: :literal:`nil`). 
	 Example: :literal:`debug = 2`.


The :literal:`correct` command returns the following object:

	:literal:`mlst`
	 A *mtable* (or a list of *mtable*) corresponding to the TFS table of the :literal:`correct` command. It is a list when multiple sequences are corrected together.


Correct mtable
--------------
.. _sec.correct.mtable:

The :literal:`correct` command returns a *mtable* where the information described hereafter is the default list of fields written to the TFS files. [#f1]_ 



	**name**
	 The name of the command that created the :literal:`"correct"`.
	**type**
	 The type of the :literal:`"correct"`.
	**title**
	 The value of the command attribute :literal:`title`.
	**origin**
	 The origin of the application that created the :literal:`"MAD 1.0.0 OSX 64"`.
	**date**
	 The date of the creation of the :literal:`"27/05/20"`.
	**time**
	 The time of the creation of the :literal:`"19:18:36"`.
	**refcol**
	 The reference *column* for the *mtable* dictionnary, e.g. :literal:`"name"`.
	**range**
	 The value of the command attribute :literal:`range`. [#f2]_ 
	**__seq**
	 The *sequence* from the command attribute :literal:`sequence`. [#f3]_ .. _ref.track.mtbl1}:



	**name**
	 The name of the element.
	**kind**
	 The kind of the element.
	**s**
	 The :math:`s`-position at the end of the element slice.
	**l**
	 The length from the start of the element to the end of the element slice.
	**x_old**
	 The local coordinate :math:`x` at the :math:`s`-position before correction.
	**y_old**
	 The local coordinate :math:`y` at the :math:`s`-position before correction.
	**x**
	 The predicted local coordinate :math:`x` at the :math:`s`-position after correction.
	**y**
	 The predicted local coordinate :math:`y` at the :math:`s`-position after correction.
	**rx**
	 The predicted local residual :math:`r_x` at the :math:`s`-position after correction.
	**ry**
	 The predicted local residual :math:`r_y` at the :math:`s`-position after correction.
	**hkick_old**
	 The local horizontal kick at the :math:`s`-position before correction.
	**vkick_old**
	 The local vertical kick at the :math:`s`-position before correction.
	**hkick**
	 The predicted local horizontal kick at the :math:`s`-position after correction.
	**vkick**
	 The predicted local vertical kick at the :math:`s`-position after correction.
	**shared**
	 A *log* indicating if the element is shared with another sequence.
	**eidx**
	 The index of the element in the sequence.

Note that :literal:`correct` does not take into account the particles and damaps :literal:`id`s present in the (augmented) :literal:`track` *mtable*, hence the provided tables should contain single particle or damap information.

Examples
--------



.. rubric:: Footnotes

.. [#f1] The output of mtable in TFS files can be fully customized by the user.
.. [#f2] This field is not saved in the TFS table by default.
.. [#f3] Fields and columns starting with two underscores are protected data and never saved to TFS files.\label{ref:track:mtbl1
