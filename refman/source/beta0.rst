Beta0 Blocks
============
.. _ch.gen.beta0:



The :literal:`beta0` object is the *root object* of beta0 blocks that store information relative to the phase space at given positions, e.g. initial conditions, Poincaré section.

The :literal:`beta0` module extends the :doc:`typeid <types>` module with the :literal:`is_beta0` *function*, which returns :literal:`true` if its argument is a :literal:`beta0` object, :literal:`false` otherwise.

Attributes
----------

The :literal:`beta0` *object* provides the following attributes:

**particle**
	 A *string* specifying the name of the particle. (default: :literal:`"positron"`).


Methods
-------

The :literal:`beta0` object provides the following methods:

**showdb**
	 A *method*	:literal:`([file])` displaying the content of the particles database to :literal:`file` (default: :literal:`io.stdout`).


Metamethods
-----------

The :literal:`beta0` object provides the following metamethods:

**__init**
	 A *metamethod*	:literal:`()` returning \SLF after having processed the attributes with the special protect-and-update mechanism, where the order of the updates is undefined. It also creates new particle on-the-fly if the :literal:`mass` and the :literal:`charge` are defined, and then select it.




**__beta0**
	 A unique private *reference* that characterizes beta0 blocks.


Examples
--------

.. rubric:: Footnotes

