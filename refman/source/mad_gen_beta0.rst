Beta0 Blocks
============
.. _ch.gen.beta0:



The :var:`beta0` object is the *root object* of beta0 blocks that store information relative to the phase space at given positions, e.g. initial conditions, Poincaré section.

The :var:`beta0` module extends the :doc:`typeid <mad_mod_types>` module with the :func:`is_beta0` *function*, which returns :const:`true` if its argument is a :var:`beta0` object, :const:`false` otherwise.

Attributes
----------

The :var:`beta0` *object* provides the following attributes:

**particle**
	 A *string* specifying the name of the particle. (default: :literal:`"positron"`).


Methods
-------

The :var:`beta0` object provides the following methods:

**showdb**
	 A *method*	:literal:`([file])` displaying the content of the particles database to :literal:`file` (default: :literal:`io.stdout`).


Metamethods
-----------

The :var:`beta0` object provides the following metamethods:

**__init**
	 A *metamethod*	:literal:`()` returning :literal:`self` after having processed the attributes with the special protect-and-update mechanism, where the order of the updates is undefined. It also creates new particle on-the-fly if the :var:`mass` and the :var:`charge` are defined, and then select it.




**__beta0**
	 A unique private *reference* that characterizes beta0 blocks.


Examples
--------

.. rubric:: Footnotes

