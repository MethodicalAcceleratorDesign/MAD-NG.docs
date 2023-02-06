Beams
=====
.. _ch.gen.beam:

The :var:`beam` object is the *root object* of beams that store information relative to particles and particle beams. It also provides a simple interface to the particles and nuclei database.

The :var:`beam` module extends the :doc:`typeid <mad_mod_types>` module with the :func:`is_beam` *function*, which returns :const:`true` if its argument is a :var:`beam` object, :const:`false` otherwise.

Attributes
----------

The :var:`beam` *object* provides the following attributes:

**particle**
	 A *string* specifying the name of the particle. (default: :literal:`"positron"`).

**mass**
	 A *number* specifying the energy-mass of the particle [GeV]. (default: :literal:`emass`).

**charge**
	 A *number* specifying the charge of the particle in [q] unit of :literal:`qelect`. [#f1]_ (default: :const:`1`).

**spin**
	 A *number* specifying the spin of the particle. (default: :const:`0`).

**emrad**
	 A *lambda* returning the electromagnetic radius of the particle [m], 

	 :math:`\mathrm{emrad} = \mathrm{krad\_GeV}\times\mathrm{charge}^2/\mathrm{mass}` where :math:`\mathrm{krad\_GeV} = 10^{-9} (4 \pi\varepsilon_0)^{-1} q`.

**aphot**
	 A *lambda* returning the average number of photon emitted per bending unit, 

	 :math:`\mathrm{aphot} = \mathrm{kpht\_GeV}\times\mathrm{charge}^2\times\mathrm{betgam}` where :math:`\mathrm{kpht\_GeV}` :math:`= \frac{5}{2\sqrt{3}}`.

**energy**
	 A *number* specifying the particle energy [GeV]. (default: :const:`1`).

**pc**
	 A *lambda* returning the particle momentum times the speed of light [GeV],

	 :math:`\mathrm{pc} = (\mathrm{energy}^2 - \mathrm{mass}^2)^{\frac{1}{2}}`.

**beta**
	 A *lambda* returning the particle relativistic :math:`\beta=\frac{v}{c}`,

	 :math:`\mathrm{beta} = (1 - (\mathrm{mass}/\mathrm{energy})^2)^{\frac{1}{2}}`.

**gamma**
	 A *lambda* returning the particle Lorentz factor :math:`\gamma=(1-\beta^2)^{-\frac{1}{2}}`,

	 :math:`\mathrm{gamma} = \mathrm{energy}/\mathrm{mass}`.

**betgam**
	 A *lambda* returning the product :math:`\beta\gamma`,

	 :math:`\mathrm{betgam} = (\mathrm{gamma}^2 - 1)^\frac{1}{2}`.

**pc2**
	 A *lambda* returning :math:`\mathrm{pc}^2`, avoiding the square root.

**beta2**
	 A *lambda* returning :math:`\mathrm{beta}^2`, avoiding the square root.

**betgam2**
	 A *lambda* returning :math:`\mathrm{betgam}^2`, avoiding the square root.

**brho**
	 A *lambda* returning the magnetic rigidity [T.m], 

	 :literal:`brho = GeV_c * pc/|charge|` where :literal:`GeV_c` = :math:`10^{9}/c`

**ex**
	 A *number* specifying the horizontal emittance :math:`\epsilon_x` [m]. (default: :const:`1`).

**ey**
	 A *number* specifying the vertical emittance :math:`\epsilon_y` [m]. (default: :const:`1`).

**et**
	 A *number* specifying the longitudinal emittance :math:`\epsilon_t` [m]. (default: :const:`1e-3`).

**exn**
	 A *lambda* returning the normalized horizontal emittance [m], 

	 :expr:`exn = ex * betgam`.

**eyn**
	 A *lambda* returning the normalized vertical emittance [m], 

	 :expr:`eyn = ey * betgam`.

**etn**
	 A *lambda* returning the normalized longitudinal emittance [m], 

	 :expr:`etn = et * betgam`.

**nbunch**
	 A *number* specifying the number of particle bunches in the machine. (default: :const:`0`).

**npart**
	 A *number* specifying the number of particles per bunch. (default: :const:`0`).

**sigt**
	 A *number* specifying the bunch length in :math:`c \sigma_t`. (default: :const:`1`).

**sige**
	 A *number* specifying the relative energy spread in :math:`\sigma_E/E` [GeV]. (default: :const:`1e-3`).


The :var:`beam` *object* also implements a special protect-and-update mechanism for its attributes to ensure consistency and precedence between the physical quantities stored internally:

*	 The following attributes are *read-only*, i.e. writing to them triggers an error:
		:literal:`mass, charge, spin, emrad, aphot`

*	 The following attributes are *read-write*, i.e. hold values, with their accepted numerical ranges:
		:literal:`particle, energy` :math:`>` :var:`mass`,
		:literal:`ex` :math:`>0`, :literal:`ey` :math:`>0`, :literal:`et` :math:`>0`,
		:literal:`nbunch` :math:`>0`, :literal:`npart` :math:`>0`, :literal:`sigt` :math:`>0`, :literal:`sige` :math:`>0`.

*	 The following attributes are *read-update*, i.e. setting these attributes update the :literal:`energy`, with their accepted numerical ranges:
		:literal:`pc` :math:`>0`, :math:`0.9>` :literal:`beta` :math:`>0`, :literal:`gamma` :math:`>1`, :literal:`betgam` :math:`>0.1`, :literal:`brho` :math:`>0`,
		:literal:`pc2`, :literal:`beta2`, :literal:`betgam2`.
*	 The following attributes are *read-update*, i.e. setting these attributes update the emittances :literal:`ex`, :literal:`ey`, and :literal:`et` repectively, with their accepted numerical ranges:
		:literal:`exn` :math:`>0`, :literal:`eyn` :math:`>0`, :literal:`etn` :math:`>0`.


Methods
-------

The :var:`beam` object provides the following methods:

**new_particle**
	 A *method*	:literal:`(particle, mass, charge, [spin])` creating new particles or nuclei and store them in the particles database. The arguments specify in order the new :literal:`particle`'s name, energy-:var:`mass` [GeV], :var:`charge` [q], and :var:`spin` (default: :const:`0`). These arguments can also be grouped into a *table* with same attribute names as the argument names and passed as the solely argument.

**set_variables**
	 A *method*	:literal:`(set)` returning :literal:`self` with the attributes set to the pairs (*key*, *value*) contained in :literal:`set`. This method overrides the original one to implement the special protect-and-update mechanism, but the order of the updates is undefined. It also creates new particle on-the-fly if the :var:`mass` and the :var:`charge` are defined, and then select it. Shortcut :literal:`setvar`.

**showdb**
	 A *method*	:literal:`([file])` displaying the content of the particles database to :literal:`file` (default: :literal:`io.stdout`).


Metamethods
-----------

The :var:`beam` object provides the following metamethods:

**__init**
	 A *metamethod*	:literal:`()` returning :literal:`self` after having processed the attributes with the special protect-and-update mechanism, where the order of the updates is undefined. It also creates new particle on-the-fly if the :var:`mass` and the :var:`charge` are defined, and then select it.

**__newindex**
	 A *metamethod*	:literal:`(key, val)` called by the assignment operator :expr:`[key]=val` to create new attributes for the pairs (*key*, *value*) or to update the underlying physical quantity of the :var:`beam` objects.


The following attribute is stored with metamethods in the metatable, but has different purpose:


**__beam**
	 A unique private *reference* that characterizes beams.


Particles database
------------------

The :var:`beam` *object* manages the particles database, which is shared by all :var:`beam` instances. The default set of supported particles is:
		electron, positron, proton, antiproton, neutron, antineutron, ion, muon, 
		antimuon, deuteron, antideuteron, negmuon (=muon), posmuon (=antimuon).

New particles can be added to the database, either explicitly using the :literal:`new_particle` method, or by creating or updating a beam *object* and specifying all the attributes of a particle, i.e. :literal:`particle`'s name, :var:`charge`, :var:`mass`, and (optional) :var:`spin`:

.. code-block:: lua
	
	local beam in MAD
	local nmass, pmass, mumass in MAD.constant
	
	-- create a new particle
	beam:new_particle{ particle='mymuon', mass=mumass, charge=-1, spin=1/2 }
	
	-- create a new beam and a new nucleus
	local pbbeam = beam { particle='pb208', mass=82*pmass+126*nmass, charge=82 }

The particles database can be displayed with the :func:`showdb` method at any time from any beam:

.. code-block:: lua
	
	beam:showdb()  -- check that both, mymuon and pb208 are in the database.


Particle charges
----------------

The physics of MAD-NG is aware of particle charges. To enable the compatibility with codes like MAD-X that ignores the particle charges, the global option :var:`nocharge` can be used to control the behavior of created beams as shown by the following example:

.. code-block:: lua
	
	local beam, option in MAD
	local beam1 = beam { particle="electron" } -- beam with negative charge
	print(beam1.charge, option.nocharge)       -- display: -1  false
	
	option.nocharge = true                     -- disable particle charges
	local beam2 = beam { particle="electron" } -- beam with negative charge
	print(beam2.charge, option.nocharge)       -- display:  1  true
	
	-- beam1 was created before nocharge activation...
	print(beam1.charge, option.nocharge)       -- display: -1  true

This approach ensures consistency of beams behavior during their entire lifetime. [#f2]_ 

Examples
--------

The following code snippet creates the LHC lead beams made of bare nuclei :math:`^{208}\mathrm{Pb}^{82+}`

.. code-block:: lua
	
	local beam in MAD
	local lhcb1, lhcb2 in MADX
	local nmass, pmass, amass in MAD.constant
	local pbmass = 82*pmass+126*nmass
	
	-- attach a new beam with a new particle to lhcb1 and lhcb2.
	lhc1.beam = beam 'Pb208' { particle='pb208', mass=pbmass, charge=82 }
	lhc2.beam = lhc1.beam -- let sequences share the same beam...
	
	-- print Pb208 nuclei energy-mass in GeV and unified atomic mass.
	print(lhcb1.beam.mass, lhcb1.beam.mass/amass)


.. rubric:: Footnotes

.. [#f1] The :var:`qelect` value is defined in the :doc:`mad_mod_const` module.
.. [#f2] The option :var:`rbarc` in MAD-X is too volatile and does not ensure such consistency...
