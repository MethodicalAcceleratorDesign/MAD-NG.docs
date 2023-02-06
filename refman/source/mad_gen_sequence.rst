#########
Sequences
#########

.. toctree::
   :numbered:

The MAD Sequences are objects convenient to describe accelerators lattices built from a *list* of elements with increasing :data:`s`-positions. The sequences are also containers that provide fast access to their elements by referring to their indexes, :data:`s`-positions, or (mangled) names, or by running iterators constrained with ranges and predicates.
The :var:`sequence` object is the *root object* of sequences that store information relative to lattices.

The :var:`sequence` module extends the :doc:`typeid <mad_mod_types>` module with the :func:`is_sequence` function, which returns :const:`true` if its argument is a :var:`sequence` object, :const:`false` otherwise.
 
Attributes
==========

The :var:`sequence` object provides the following attributes: 

**l**
   A *number* specifying the length of the sequence :literal:`[m]`. A :const:`nil` will be replaced by the computed lattice length. A value greater or equal to the computed lattice length will be used to place the :literal:`$end` marker. Other values will raise an error. (default: :const:`nil`).
 
**dir**
   A *number* holding one of :const:`1` (forward) or :const:`-1` (backward) and specifying the direction of the sequence. [#f1]_ (default:~ :const:`1`)

**refer** 
   A *string* holding one of :literal:`"entry"`, :literal:`"centre"` or return true :literal:`"exit"` to specify the default reference position in the elements to use for their placement. An element can override it with its :literal:`refpos` attribute, see `element positions`_ for details. (default: :const:`nil` :math:`\equiv` :literal:`"centre"`).

**minlen**
   A *number* specifying the minimal length :literal:`[m]` when checking for negative drifts or when generating *implicit* drifts between elements in :math:`s`-iterators returned by the method :literal:`:siter`. This attribute is automatically set to :math:`10^{-6}` m when a sequence is created within the MADX environment. (default: :math:`10^{-6}`)

**beam** 
   An attached :var:`beam`. (default: :const:`nil`)

**Warning**: the following private and read-only attributes are present in all sequences and should *never be used, set or changed*; breaking this rule would lead to an *undefined behavior*:

**__dat**
   A *table* containing all the private data of sequences.

**__cycle**
   A *reference* to the element registered with the :literal:`:cycle` method. (default: :const:`nil`)


Methods
=======

The :var:`sequence` object provides the following methods:

**elem** 
   A *method* :literal:`(idx)` returning the element stored at the positive index :literal:`idx` in the sequence, or :const:`nil`.

**spos**
   A *method* :literal:`(idx)` returning the :math:`s`-position at the entry of the element stored at the positive index :literal:`idx` in the sequence, or :const:`nil`.

**upos**
   A *method* :literal:`(idx)` returning the :math:`s`-position at the user-defined :literal:`refpos` offset of the element stored at the positive index :literal:`idx` 
   in the sequence, or :const:`nil`.

**ds**
   A *method* :literal:`(idx)` returning the length of the element stored at the positive index :literal:`idx` in the sequence, or :const:`nil`.

**align**
   A *method* :literal:`(idx)` returning a *set* specifying the misalignment of the element stored at the positive index :literal:`idx` in the sequence, or :const:`nil`.

**index**
   A *method* :literal:`(idx)` returning a positive index, or :const:`nil`. If :literal:`idx` is negative, it is reflected versus the size of the sequence, e.g. :const:`-1` 
   becomes :literal:`#self`, the index of the :literal:`$end` marker.

**name_of**
   A *method* :literal:`(idx, [ref])` returning a *string* corresponding to the (mangled) name of the element at the index :literal:`idx` or :const:`nil`. An element 
   name appearing more than once in the sequence will be mangled with an absolute count, e.g. :literal:`mq[3]`, or a relative count versus the optional 
   reference element :literal:`ref` determined by :literal:`:index_of`, e.g. :literal:`mq{-2}`.

**index_of**
   A *method* :literal:`(a, [ref], [dir])` returning a *number* corresponding to the positive index of the element determined by the first argument or :const:`nil`. 
   If :literal:`a` is a *number* (or a *string* representing a *number*), it is interpreted as the :math:`s`-position of an element and returned as a second 
   *number*. If :literal:`a` is a *string*, it is interpreted as the (mangled) name of an element as returned by :literal:`:name_of`. Finally, :literal:`a` can be a *reference* 
   to an element to search for. The argument :literal:`ref` (default: :literal:`nil)` specifies the reference element determined by :literal:`:index_of(ref)` to use for 
   relative :math:`s`-positions, for decoding mangled names with relative counts, or as the element to start searching from. The argument :literal:`dir` 
   (default: :literal:`1)` specifies the direction of the search with values :const:`1` (forward), :const:`-1` (backward), or :const:`0` (no direction). The :literal:`dir=0`
   case may return an index at half-integer if :literal:`a` is interpreted as an :math:`s`-position pointing to an *implicit drift*.

**range_of**
   A *method* :literal:`([rng], [ref], [dir]`) returning three *numbers* corresponding to the positive indexes *start* and *end* of the range and 
   its direction *dir*, or :const:`nil` for an empty range. If :literal:`rng` is omitted, it returns :const:`1`, :literal:`#self`, :const:`1`, or :literal:`#self`, :const:`1`, :const:`-1` 
   if :literal:`dir` is negative. If :literal:`rng` is a *number* or a *string* with no :literal:`'/'` separator, it is interpreted as both *start* and *end* and 
   determined by :literal:`index_of`. If :literal:`rng` is a *string* containing the separator :literal:`'/'`, it is split in two *strings* interpreted as *start* 
   and *end*, both determined by :literal:`:index_of`. If :literal:`rng` is a *list*, it will be interpreted as {*start*, *end*, :literal:`[ref]`, :literal:`[dir]`}, 
   both determined by :literal:`:index_of`, unless :literal:`ref` equals :literal:`'idx'` then both are determined by :literal:`:index` (i.e. a *number* is interpreted as an 
   index instead of a :math:`s`-position). The arguments :literal:`ref` (default: :const:`nil`) and :literal:`dir` (default: :const:`1`) are forwarded to all invocations 
   of :literal:`:index_of` with a higher precedence than ones in the *list* :literal:`rng`, and a runtime error is raised if the method returns :const:`nil`, i.e. 
   to disambiguate between a valid empty range and an invalid range.

**length_of**
   A *method* :literal:`([rng], [ntrn], [dir]`) returning a *number* specifying the length of the range optionally including :literal:`ntrn` extra turns (default: :const:`0`), 
   and calculated from the indexes returned by :literal:`:range_of([rng], nil, [dir])`.

**iter**
   A *method* :literal:`([rng], [ntrn], [dir])` returning an iterator over the sequence elements. The optional range is determined by 
   :meth:`:range_of(rng, [dir])`, optionally including :literal:`ntrn` turns (default: :const:`0`). The optional direction :literal:`dir` specifies the forward :const:`1` 
   or the backward :const:`-1` direction of the iterator. If :literal:`rng` is not provided and the ?sequence? is cycled, the *start* and *end* indexes are 
   determined by :literal:`:index_of(self.__cycle)`. When used with a generic :literal:`for` loop, the iterator returns at each element: its index, 
   the element itself, its :math:`s`-position over the running loop and its signed length depending on the direction.

**siter**
   A *method* :literal:`([rng], [ntrn], [dir])` returning an :math:`s`-iterator over the sequence elements. The optional range is determined by 
   :literal:`:range_of([rng], nil, [dir])`, optionally including :literal:`ntrn` turns (default: :const:`0`). The optional direction :literal:`dir` specifies the 
   forward :const:`1` or the backward :const:`-1` direction of the iterator. When used with a generic :literal:`for` loop, the iterator returns at each 
   iteration: its index, the element itself or an *implicit* :literal:`drift`, its :math:`s`-position over the running loop and its signed length 
   depending on the direction. Each *implicit* drift is built on-the-fly by the iterator with a length equal to the gap between the elements 
   surrounding it and a half-integer index equal to the average of their indexes. The length of *implicit* drifts is bounded by the maximum 
   between the sequence attribute :literal:`minlen` and the :literal:`minlen` from the :doc:`constant <mad_mod_const>` module.

**foreach**
   A *method* :literal:`(act, [rng], [sel], [not])` returning the sequence itself after applying the action :literal:`act` on the selected elements. If :literal:`act` 
   is a *set* representing the arguments in the packed form, the missing arguments will be extracted from the attributes :literal:`action`, 
   :literal:`range`, :literal:`select` and :literal:`default`. The action :literal:`act` must be a *callable* :literal:`(elm, idx, [midx])` applied to an element passed as 
   first argument and its index as second argument, the optional third argument being the index of the main element in case :literal:`elm` is a sub-element. 
   The optional range is used to generate the loop iterator :literal:`:iter([rng])`. The optional selector :literal:`sel` is a *callable* :literal:`(elm, idx, [midx])`
   predicate selecting eligible elements for the action using the same arguments. The selector :literal:`sel` can be specified in other ways, 
   see `element selections`_ for details. The optional *logical* :literal:`not` (default: :const:`false`) indicates how to interpret default selection, as 
   *all* or *none*, depending on the semantic of the action. [#f2]_
**select**
   A *method* :literal:`([flg], [rng], [sel], [not])` returning the sequence itself after applying the action :literal:`:select([flg])` to the elements using
   :literal:`:foreach(act, [rng], [sel], [not])`. By default sequence have all their elements deselected with only the :literal:`$end` marker :literal:`observed`.

**deselect**
   A *method* :literal:`([flg], [rng], [sel], [not])` returning the sequence itself after applying the action :literal:`:deselect([flg])` to the elements 
   using :literal:`:foreach(act, [rng], [sel], [not])`. By default sequence have all their elements deselected with only the :literal:`$end` marker :literal:`observed`.

**filter**
   A *method* :literal:`([rng], [sel], [not])` returning a *list* containing the positive indexes of the elements determined by :literal:`:foreach(act, [rng], [sel], [not])`,
   and its size. The *logical* :literal:`sel.subelem` specifies to select sub-elements too, and the *list* may contain non-integer indexes encoding their main element 
   index added to their relative position, i.e. :literal:`midx.sat`. The builtin *function* :literal:`math.modf(num)` allows to retrieve easily the main element :literal:`midx` and
   the sub-element :literal:`sat`, e.g. :literal:`midx,sat = math.modf(val)`.

**install**
   A *method* :literal:`(elm, [rng], [sel], [cmp])` returning the sequence itself after installing the elements in the *list* :literal:`elm` at their 
   `element positions`_; unless :literal:`from="selected"` is defined meaning multiple installations at positions relative to each element determined by the method
   :literal:`:filter([rng], [sel], true)`. The *logical* :literal:`sel.subelem` is ignored. If the arguments are passed in the packed form, the extra attribute :literal:`elements` 
   will be used as a replacement for the argument :literal:`elm`. The *logical* :literal:`elm.subelem` specifies to install elements with :math:`s`-position falling inside 
   sequence elements as sub-elements, and set their :literal:`sat` attribute accordingly. The optional *callable* :literal:`cmp(elmspos, spos[idx])` (default: :literal:`"<"`) is used
   to search for the :math:`s`-position of the installation, where equal :math:`s`-position are installed after (i.e. before with :literal:`"<="`), see :literal:`bsearch` from
   the :doc:`miscellaneous <mad_mod_miscfuns>` module for details. The *implicit* drifts are checked after each element installation.

**replace**
   A *method* :literal:`(elm, [rng], [sel])` returning the *list* of replaced elements by the elements in the *list* :literal:`elm` placed at their `element positions`_, and the
   *list* of their respective indexes, both determined by :literal:`:filter([rng], [sel], true)`. The *list* :literal:`elm` cannot contain instances of :var:`sequence` or :literal:`bline`
   elements and will be recycled as many times as needed to replace all selected elements. If the arguments are passed in the packed form, the extra attribute
   :literal:`elements` will be used as a replacement for the argument :literal:`elm`. The *logical* :literal:`sel.subelem` specifies to replace selected sub-elements too and set
   their :literal:`sat` attribute to the same value. The *implicit* drifts are checked only once all elements have been replaced.

**remove**
   A *method* :literal:`([rng], [sel])` returning the *list* of removed elements and the *list* of their respective indexes, both determined by :literal:`:filter([rng], [sel], true)`.
   The *logical* :literal:`sel.subelem` specifies to remove selected sub-elements too.

**move**
   A *method* :literal:`([rng], [sel])` returning the sequence itself after updating the `element positions`_ at the indexes determined by :literal:`:filter([rng], [sel], true)`. 
   The *logical* :literal:`sel.subelem` is ignored. The elements must keep their order in the sequence and surrounding *implicit* drifts are checked only once all elements
   have been moved. [#f3]_

**update**
   A *method* :literal:`()` returning the sequence itself after recomputing the positions of all elements.

**misalign**
   A *method* :literal:`(algn, [rng], [sel])` returning the sequence itself after setting the :ref:`element misalignments <sec.elm.misalign>` from 
   :literal:`algn` at the indexes determined by :literal:`:filter([rng], [sel], true)`. If :literal:`algn` is a *mappable*, it will be used to misalign the filtered elements. 
   If :literal:`algn` is a *iterable*, it will be accessed using the filtered elements indexes to retrieve their specific misalignment. 
   If :literal:`algn` is a *callable* :literal:`(idx)`, it will be invoked for each filtered element with their index as solely argument to retrieve their specific misalignment.

**reflect**
   A *method* :literal:`([name])` returning a new sequence from the sequence reversed, and named from the optional *string* :literal:`name` (default: :literal:`self.name..'_rev'`).

**cycle**
   A *method* :literal:`(a)` returning the sequence itself after checking that :literal:`a` is a valid reference using :literal:`:index_of(a)`, and storing it in the :literal:`__cycle` attribute, 
   itself erased by the methods editing the sequence like :literal:`:install`, :literal:`:replace`, :literal:`:remove`, :literal:`:share`, and :literal:`:unique`.

**share**
   A *method* :literal:`(seq2)` returning the *list* of elements removed from the :literal:`seq2` and the *list* of their respective indexes, and replaced by the elements from the
   sequence with the same name when they are unique in both sequences.

**unique**
   A *method* :literal:`([fmt])` returning the sequence itself after replacing all non-unique elements by new instances sharing the same parents. 
   The optional :literal:`fmt` must be a *callable* :literal:`(name, cnt, idx)` that returns the mangled name of the new instance build from the element :literal:`name`,
   its count :literal:`cnt` and its index :literal:`idx` in the sequence. If the optional :literal:`fmt` is a *string*, the mangling *callable* is built by binding :literal:`fmt` 
   as first argument to the function :literal:`string.format` from the standard library, see 
   `Lua 5.2 <http://github.com/MethodicalAcceleratorDesign/MADdocs/blob/master/lua52-refman-madng.pdf>`_ §6.4 for details.

**publish**
   A *method* :literal:`(env, [keep])` returning the sequence after publishing all its elements in the environment :literal:`env`. If the *logical* :literal:`keep` is
   :const:`true`, the method will preserve existing elements from being overridden. This method is automatically invoked with :literal:`keep=true` when sequences
   are created within the :literal:`MADX` environment.

**copy**
   A *method* :literal:`([name], [owner])` returning a new sequence from a copy of :literal:`self`, with the optional :literal:`name` and the optional attribute :literal:`owner` set. 
   If the sequence is a view, so will be the copy unless :literal:`owner == true`.

**set_readonly**
   Set the sequence as read-only, including its columns.

**save_flags**
   A *method* :literal:`([flgs])` saving the flags of all the elements to the optional *iterable* :literal:`flgs` (default: :literal:`{}`) and return it.

**restore_flags**
   A *method* :literal:`(flgs)` restoring the flags of all the elements from the *iterable* :literal:`flgs`. The indexes of the flags must match the indexes of the elements
   in the sequence.

**dumpseq**
   A *method* :literal:`([fil], [info])` displaying on the optional file :literal:`fil` (default: :literal:`io.stdout`) information related to the position and length of the elements.
   Useful to identify negative drifts and badly positioned elements. The optional argument :literal:`info` indicates to display extra information like elements misalignments.

**check_sequ**
   A *method* () checking the integrity of the sequence and its dictionary, for debugging purpose only.


Metamethods
===========

The :var:`sequence` object provides the following metamethods:

**__len** 
   A *method* () called by the length operator :literal:`#` to return the size of the sequence, i.e. the number of elements stored including the :literal:`"\$start"` and 
   :literal:`"\$end"` markers.

**__index** 
   A *method* :literal:`(key)` called by the indexing operator :literal:`[key]` to return the *value* of an attribute determined by *key*. The *key* is interpreted differently depending 
   on its type with the following precedence:
   1. A *number* is interpreted as an element index and returns the element or :const:`nil`.
   #. Other *key* types are interpreted as *object* attributes subject to object model lookup.
   #. If the *value* associated with *key* is :const:`nil`, then *key* is interpreted as an element name and returns either the element or an *iterable* on the elements with the same name. [#f4]_
   #. Otherwise returns :const:`nil`.

**__newindex**
   A *method* :literal:`(key, val)` called by the assignment operator :literal:`[key]=val` to create new attributes for the pairs (*key*, *value*). 
   If *key* is a *number* specifying the index or a *string* specifying the name of an existing element, the following error is raised:
   :literal:`"invalid sequence write access (use replace method)"`


**__init**
   A *method* () called by the constructor to compute the elements positions.

**__copy**
   A *method* () similar to the :literal:`:copy` *method*.

The following attribute is stored with metamethods in the metatable, but has different purpose:

**__sequ** A unique private *reference* that characterizes sequences.

Sequences creation
==================

During its creation as an *object*, a sequence can defined its attributes as any object, and the *list* of its elements that must form a
*sequence* of increasing :math:`s`-positions. When subsequences are part of this *list*, they are replaced by their respective elements as a 
sequence *element* cannot be present inside other sequences. If the length of the sequence is not provided, it will be computed and set automatically. 
During their creation, sequences compute the :math:`s`-positions of their elements as described in the section `element positions`_, and check for overlapping
elements that would raise a "negative drift" runtime error.

The following example shows how to create a sequence form a *list* of elements and subsequences:

::

   local sequence, drift, marker in MAD.element
   local df, mk = drift 'df' {l=1}, marker 'mk' {}
   local seq = sequence 'seq' {
   df 'df1' {}, mk 'mk1' {},
   sequence {
      sequence { mk 'mk0' {} },
      df 'df.s' {}, mk 'mk.s' {}
   },
   df 'df2' {}, mk 'mk2' {},
   } :dumpseq()

Displays

.. code-block:: text

   sequence: seq, l=3
   idx  kind     name         l          dl       spos       upos    uds
   001  marker   (*$start*)   0.000       0       0.000      0.000   0.000
   002  drift    df1          1.000       0       0.000      0.500   0.500
   003  marker   mk1          0.000       0       1.000      1.000   0.000
   004  marker   mk0          0.000       0       1.000      1.000   0.000
   005  drift    df.s         1.000       0       1.000      1.500   0.500
   006  marker   mk.s         0.000       0       2.000      2.000   0.000
   007  drift    df2          1.000       0       2.000      2.500   0.500
   008  marker   mk2          0.000       0       3.000      3.000   0.000
   009  marker   (*$end*)     0.000       0       3.000      3.000   0.000

.. _elpos:

Element positions
=================

A sequence looks at the following attributes of an element, including sub-sequences, when installing it, *and only at that time*, to determine its position:

**at**
   A *number* holding the position in [m] of the element in the sequence relative to the position specified by the :literal:`from` attribute.

**from**
   A *string* holding one of :literal:`"start"`, :literal:`"prev"`, :literal:`"next"`, :literal:`"end"` or :literal:`"selected"`, or the (mangled) name of another element to use as the reference position,
   or a *number* holding a position in [m] from the start of the sequence. (default: :literal:`"start"` if :literal:`at`:math:`\geq 0`, :literal:`"end"` if :literal:`at`:math:`<0`, and :literal:`"prev"` 
   otherwise)

**refpos**
   A *string* holding one of :literal:`"entry"`, :literal:`"centre"` or :literal:`"exit"`,  or the (mangled) name of a sequence sub-element to use as the reference position,
   or a *number* specifying a position [m] from the start of the element, all of them resulting in an offset to substract to the :literal:`at` attribute to find the 
   :math:`s`-position of the element entry. (default: :const:`nil` :math:`\equiv` :literal:`self.refer`).

**shared** 
   A *logical* specifying if an element is used at different positions in the same sequence definition, i.e. shared multiple times,
   through temporary instances to store the many :literal:`at` and :literal:`from` attributes needed to specify its positions. 
   Once built, the sequence will drop these temporary instances in favor of their common parent, i.e. the original shared element.

**Warning:** 
   The :literal:`at` and :literal:`from` attributes are not considered as intrinsic properties of the elements and are used only once during installation.
   Any reuse of these attributes is the responsibility of the user, including the consistency between :literal:`at` and :literal:`from` after updates.


Element selections
==================

The element selection in sequence use predicates in combination with iterators. The sequence iterator manages the range of elements where to apply the selection, 
while the predicate says if an element in this range is illegible for the selection. In order to ease the use of methods based on the :literal:`:foreach` method, 
the selector predicate :literal:`sel` can be built from different types of information provided in a *set* with the following attributes:

**flag**
   A *number* interpreted as a flags mask to pass to the element method :literal:`:is_selected`. It should not be confused with the flags passed as argument to methods
   :literal:`:select` and :literal:`:deselect`, as both flags can be used together but with different meanings!

**pattern**
   A *string* interpreted as a pattern to match the element name using :literal:`string.match` from the standard library, see
   `Lua 5.2 <http://github.com/MethodicalAcceleratorDesign/MADdocs/blob/master/lua52-refman-madng.pdf>`_ §6.4 for details.

**class**
   An *element* interpreted as a *class* to pass to the element method :literal:`:is_instansceOf`.

**list**
   An *iterable* interpreted as a *list* used to build a *set* and select the elements by their name, i.e. the built predicate will use :literal:`tbl[elm.name]` 
   as a *logical*. If the *iterable* is a single item, e.g. a *string*, it will be converted first to a *list*.

**table** 
   A *mappable* interpreted as a *set* used to select the elements by their name, i.e. the built predicate will use :literal:`tbl[elm.name]` as a *logical*. 
   If the *mappable* contains a *list* or is a single item, it will be converted first to a *list* and its *set* part will be discarded.

**select** 
   A *callable* interpreted as the selector itself, which allows to build any kind of predicate or to complete the restrictions already built above.

**subelem** 
   A *boolean* indicating to include or not the sub-elements in the scanning loop. The predicate and the action receive the sub-element and its sub-index as
   first and second argument, and the main element index as third argument.

All these attributes are used in the aforementioned order to incrementally build predicates that are combined with logical conjunctions, i.e. :literal:`and`'ed, 
to give the final predicate used by the :literal:`:foreach` method. If only one of these attributes is needed, it is possible to pass it directly in :literal:`sel`, 
not as an attribute in a *set*, and its type will be used to determine the kind of predicate to build. For example, :literal:`self:foreach(act, monitor)` is equivalent
to :literal:`self:foreach\{action=act, class=monitor}`.

Indexes, names and counts
=========================

Indexing a sequence triggers a complex look up mechanism where the arguments will be interpreted in various ways as described in the :literal:`:__index` metamethod. 
A *number* will be interpreted as a relative slot index in the list of elements, and a negative index will be considered as relative to the end of the sequence, 
i.e. :const:`-1` is the :literal:`$end` marker. Non- *number* will be interpreted first as an object key (can be anything), looking for sequence methods or attributes;
then as an element name if nothing was found.

If an element exists but its name is not unique in the sequence, an *iterable* is returned. An *iterable* supports the length :literal:`#` operator to retrieve the 
number of elements with the same name, the indexing operator :literal:`[]` waiting for a count $n$ to retrieve the :math:`n`-th element from the start with that name,
and the iterator :literal:`ipairs` to use with generic :literal:`for` loops.

The returned *iterable* is in practice a proxy, i.e. a fake intermediate object that emulates the expected behavior, and any attempt to access the proxy in 
another manner should raise a runtime error.

**Warning:** The indexing operator :literal:`[]` interprets a *number* as a (relative) element index as the method :literal:`:index`, while the method :literal:`:index_of` interprets a 
*number* as a (relative) element :math:`s`-position [m].

The following example shows how to access to the elements through indexing and the *iterable*:::

   local sequence, drift, marker in MAD.element
   local seq = sequence {
   drift 'df' { id=1 }, marker 'mk' { id=2 },
   drift 'df' { id=3 }, marker 'mk' { id=4 },
   drift 'df' { id=5 }, marker 'mk' { id=6 },
   }
   print(seq[ 1].name) -- display: (*\$start*) (start marker)
   print(seq[-1].name) -- display: (*\$end*)   (end   marker)

   print(#seq.df, seq.df[3].id)                        -- display: 3   5
   for _,e in ipairs(seq.df) do io.write(e.id," ") end -- display: 1 3 5
   for _,e in ipairs(seq.mk) do io.write(e.id," ") end -- display: 2 4 6

   -- print name of drift with id=3 in absolute and relative to id=6.
   print(seq:name_of(4))       -- display: df[2]  (2nd df from start)
   print(seq:name_of(2, -2))   -- display: df{-3} (3rd df before last mk)


The last two lines of code display the name of the same element but mangled with absolute and relative counts.

Iterators and ranges
====================

Ranging a sequence triggers a complex look up mechanism where the arguments will be interpreted in various ways as described in the :literal:`:range_of` method, 
itself based on the methods :literal:`:index_of` and :literal:`:index`. The number of elements selected by a sequence range can be computed by the :literal:`:length_of` method,
which accepts an extra *number* of turns to consider in the calculation.

The sequence iterators are created by the methods :literal:`:iter` and :literal:`:siter`, and both are based on the :literal:`:range_of` method as mentioned in their descriptions
and includes an extra *number* of turns as for the method :literal:`:length_of`, and a direction :const:`1` (forward) or :const:`-1` (backward) for the iteration.
The :literal:`:siter` differs from the :literal:`:iter` by its loop, which returns not only the sequence elements but also *implicit* drifts built on-the-fly when a gap 
:math:`>10^{-10}` m is detected between two sequence elements. Such implicit drift have half-integer indexes and make the iterator "continuous" in :math:`s`-positions.

The method :literal:`:foreach` uses the iterator returned by :literal:`:iter` with a range as its sole argument to loop over the elements where to apply the predicate before 
executing the action. The methods :literal:`:select`, :literal:`:deselect`, :literal:`:filter`, :literal:`:install`, :literal:`:replace`, :literal:`:remove`, :literal:`:move`, and :literal:`:misalign` are all based
directly or indirectly on the :literal:`:foreach` method. Hence, to iterate backward over a sequence range, these methods have to use either its *list* form or a numerical range.
For example the invocation :literal:`seq:foreach(\e -> print(e.name), {2, 2, 'idx', -1)` will iterate backward over the entire sequence :literal:`seq` excluding the :literal:`$start`
and :literal:`$end` markers, while the invocation :literal:`seq:foreach(\e -> print(e.name), 5..2..-1)` will iterate backward over the elements with :math:`s`-positions sitting in the
interval :math:`[2,5]` m.

The tracking commands :literal:`survey` and :var:`track` use the iterator returned by :literal:`:siter` for their main loop, with their :literal:`range`, :literal:`nturn` and :literal:`dir` attributes as arguments. These commands also save the iterator states in their :literal:`mflw` to allow the users to run them :literal:`nstep` by :literal:`nstep`, see commands :doc:`survey <mad_cmd_survey>` and :doc:`track <mad_cmd_track>` for details.

The following example shows how to access to the elements with the :literal:`:foreach` method:::

   local sequence, drift, marker in MAD.element
   local observed in MAD.element.flags
   local seq = sequence {
   drift 'df' { id=1 }, marker 'mk' { id=2 },
   drift 'df' { id=3 }, marker 'mk' { id=4 },
   drift 'df' { id=5 }, marker 'mk' { id=6 },
   }

   local act = \e -> print(e.name,e.id)
   seq:foreach(act, "df[2]/mk[3]")
   -- display:
   df   3
   mk   4
   df   5
   mk   6

   seq:foreach{action=act, range="df[2]/mk[3]", class=marker}
   -- display: markers at ids 4 and 6
   seq:foreach{action=act, pattern=(*\verb+"^[^$]"+*)}
   -- display: all elements except (*\verb+$start and $end+*) markers
   seq:foreach{action=\e -> e:select(observed), pattern="mk"}
   -- same as: seq:select(observed, {pattern="mk"})

   local act = \e -> print(e.name, e.id, e:is_observed())
   seq:foreach{action=act, range=(*\verb+"#s/#e"+*)}
   -- display:
   (*\$start*)   nil  false
   df       1    false
   mk       2    true
   df       3    false
   mk       4    true
   df       5    false
   mk       6    true
   (*\$end*)     nil  true

Examples
========

FODO cell
---------

.. code-block::
   
   local sequence, sbend, quadrupole, sextupole, hkicker, vkicker, marker in MAD.element
   local mkf = marker 'mkf' {}
   local ang=2*math.pi/80
   local fodo = sequence 'fodo' { refer='entry',
   mkf             { at=0, shared=true      }, -- mark the start of the fodo
   quadrupole 'qf' { at=0, l=1  , k1=0.3    },
   sextupole  'sf' {       l=0.3, k2=0      },
   hkicker    'hk' {       l=0.2, kick=0    },
   sbend      'mb' { at=2, l=2  , angle=ang },

   quadrupole 'qd' { at=5, l=1  , k1=-0.3   },
   sextupole  'sd' {       l=0.3, k2=0      },
   vkicker    'vk' {       l=0.2, kick=0    },
   sbend      'mb' { at=7, l=2  , angle=ang },
   }
   local arc = sequence 'arc' { refer='entry', 10*fodo }
   fodo:dumpseq() ; print(fodo.mkf, mkf)


Display:

.. code-block:: text

   sequence: fodo, l=9
   idx  kind          name          l          dl       spos       upos    uds
   001  marker        $start  0.000       0       0.000      0.000   0.000
   002  marker        mkf     0.000       0       0.000      0.000   0.000
   003  quadrupole    qf      1.000       0       0.000      0.000   0.000
   004  sextupole     sf      0.300       0       1.000      1.000   0.000
   005  hkicker       hk      0.200       0       1.300      1.300   0.000
   006  sbend         mb      2.000       0       2.000      2.000   0.000
   007  quadrupole    qd      1.000       0       5.000      5.000   0.000
   008  sextupole     sd      0.300       0       6.000      6.000   0.000
   009  vkicker       vk      0.200       0       6.300      6.300   0.000
   010  sbend         mb      2.000       0       7.000      7.000   0.000
   011  marker        $end    0.000       0       9.000      9.000   0.000
   marker : 'mkf' 0x01015310e8	marker: 'mkf' 0x01015310e8 -- same marker


SPS compact description
-----------------------

The following dummy example shows a compact definition of the SPS mixing elements, beam lines and sequence definitions.
The elements are zero-length, so the lattice is too. ::

   local drift, sbend, quadrupole, bline, sequence in MAD.element

   -- elements (empty!)
   local ds = drift      'ds' {}
   local dl = drift      'dl' {}
   local dm = drift      'dm' {}
   local b1 = sbend      'b1' {}
   local b2 = sbend      'b2' {}
   local qf = quadrupole 'qf' {}
   local qd = quadrupole 'qd' {}

   -- subsequences
   local pf  = bline 'pf'  {qf,2*b1,2*b2,ds}           -- #: 6
   local pd  = bline 'pd'  {qd,2*b2,2*b1,ds}           -- #: 6
   local p24 = bline 'p24' {qf,dm,2*b2,ds,pd}          -- #: 11 (5+6)
   local p42 = bline 'p42' {pf,qd,2*b2,dm,ds}          -- #: 11 (6+5)
   local p00 = bline 'p00' {qf,dl,qd,dl}               -- #: 4
   local p44 = bline 'p44' {pf,pd}                     -- #: 12 (6+6)
   local insert = bline 'insert' {p24,2*p00,p42}       -- #: 30 (11+2*4+11)
   local super  = bline 'super'  {7*p44,insert,7*p44}  -- #: 198 (7*12+30+7*12)

   -- final sequence
   local SPS = sequence 'SPS' {6*super}                -- # = 1188 (6*198)

   -- check number of elements and length
   print(#SPS, SPS.l)  -- display: 1190  0 (no element length provided)


Installing elements I
---------------------

The following example shows how to install elements and subsequences in an empty initial sequence:::

   local sequence, drift in MAD.element
   local seq   = sequence "seq" { l=16, refer="entry", owner=true }
   local sseq1 = sequence "sseq1" {
   at=5, l=6 , refpos="centre", refer="entry",
   drift "df1'" {l=1, at=-4, from="end"},
   drift "df2'" {l=1, at=-2, from="end"},
   drift "df3'" {     at= 5            },
   }
   local sseq2 = sequence "sseq2" {
   at=14, l=6, refpos="exit", refer="entry",
   drift "df1''" { l=1, at=-4, from="end"},
   drift "df2''" { l=1, at=-2, from="end"},
   drift "df3''" {      at= 5            },
   }
   seq:install {
   drift "df1" {l=1, at=1},
   sseq1, sseq2,
   drift "df2" {l=1, at=15},
   } :dumpseq()

Display:

.. code-block:: text

   sequence: seq, l=16
   idx  kind          name       l          dl       spos       upos    uds
   001  marker        $start*    0.000       0       0.000      0.000   0.000
   002  drift         df1        1.000       0       1.000      1.000   0.000
   003  drift         df1'       1.000       0       4.000      4.000   0.000
   004  drift         df2'       1.000       0       6.000      6.000   0.000
   005  drift         df3'       0.000       0       7.000      7.000   0.000
   006  drift         df1''      1.000       0      10.000     10.000   0.000
   007  drift         df2''      1.000       0      12.000     12.000   0.000
   008  drift         df3''      0.000       0      13.000     13.000   0.000
   009  drift         df2        1.000       0      15.000     15.000   0.000
   010  marker        $end       0.000       0      16.000     16.000   0.000

Installing elements II
----------------------

The following more complex example shows how to install elements and subsequences in a sequence using a selection and the packed form for arguments:::

   local mk   = marker   "mk"  { }
   local seq  = sequence "seq" { l = 10, refer="entry",
   mk "mk1" { at = 2 },
   mk "mk2" { at = 4 },
   mk "mk3" { at = 8 },
   }
   local sseq = sequence "sseq" { l = 3 , at = 5, refer="entry",
   drift "df1'" { l = 1, at = 0 },
   drift "df2'" { l = 1, at = 1 },
   drift "df3'" { l = 1, at = 2 },
   }
   seq:install {
   class    = mk,
   elements = {
      drift "df1" { l = 0.1, at = 0.1, from="selected" },
      drift "df2" { l = 0.1, at = 0.2, from="selected" },
      drift "df3" { l = 0.1, at = 0.3, from="selected" },
      sseq,
      drift "df4" { l = 1, at = 9 },
   }
   }

   seq:dumpseq()

.. code-block:: text
   
   sequence: seq, l=10
   idx  kind          name      l          dl       spos       upos    uds
   001  marker        $start    0.000       0       0.000      0.000   0.000
   002  marker        mk1       0.000       0       2.000      2.000   0.000
   003  drift         df1       0.100       0       2.100      2.100   0.000
   004  drift         df2       0.100       0       2.200      2.200   0.000
   005  drift         df3       0.100       0       2.300      2.300   0.000
   006  marker        mk2       0.000       0       4.000      4.000   0.000
   007  drift         df1       0.100       0       4.100      4.100   0.000
   008  drift         df2       0.100       0       4.200      4.200   0.000
   009  drift         df3       0.100       0       4.300      4.300   0.000
   010  drift         df1'      1.000       0       5.000      5.000   0.000
   011  drift         df2'      1.000       0       6.000      6.000   0.000
   012  drift         df3'      1.000       0       7.000      7.000   0.000
   013  marker        mk3       0.000       0       8.000      8.000   0.000
   014  drift         df1       0.100       0       8.100      8.100   0.000
   015  drift         df2       0.100       0       8.200      8.200   0.000
   016  drift         df3       0.100       0       8.300      8.300   0.000
   017  drift         df4       1.000       0       9.000      9.000   0.000
   018  marker        $end      0.000       0      10.000     10.000   0.000


.. rubric:: Footnotes

.. [#f1] This is equivalent to the MAD-X :literal:`bv` flag.
.. [#f2] For example, the :literal:`:remove` method needs :literal:`not=true` to *not* remove all elements if no selector is provided.
.. [#f3] Updating directly the positions attributes of an element has no effect.
.. [#f4] An *iterable* supports the length operator :literal:`#`, the indexing operator :literal:`[]` and generic :literal:`for` loops with :literal:`ipairs`.
.. [#f5] MAD-NG does not have a MAD-X like :literal:`"USE"` command to finalize this computation.
