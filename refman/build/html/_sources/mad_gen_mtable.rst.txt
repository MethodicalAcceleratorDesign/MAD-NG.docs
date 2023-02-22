MTables
=======
.. _ch.gen.mtbl:

The MAD Tables (MTables) --- also named Table File System (TFS) --- are objects convenient to store, read and write a large amount of heterogeneous information organized as columns and header. The MTables are also containers that provide fast access to their rows, columns, and cells by referring to their indexes, or some values of the designated reference column, or by running iterators constrained with ranges and predicates.

The :literal:`mtable` object is the *root object* of the TFS tables that store information relative to tables.

The :literal:`mtable` module extends the :doc:`typeid <mad_mod_types>` module with the :func:`is_mtable` function, which returns :const:`true` if its argument is a :literal:`mtable` object, :const:`false` otherwise.

Attributes
----------

The :literal:`mtable` object provides the following attributes:

**type**
	 A *string* specifying the type of the mtable (often) set to the name of the command that created it, like :literal:`survey`, :var:`track` or :var:`twiss`. (default: :literal:`'user'`).

**title**
	 A *string* specifying the title of the mtable (often) set to the attribute :literal:`title` of the command that created it. (default: :literal:`'no-title'`).

**origin**
	 A *string* specifying the origin of the mtable. (default: :literal:`"MAD version os arch"`).

**date**
	 A *string* specifying the date of creation of the mtable. (default: :literal:`"day/month/year"`).

**time**
	 A *string* specifying the time of creation of the mtable. (default: :literal:`"hour:min:sec"`).

**refcol**
	 A *string* specifying the name of the reference column used to build the dictionary of the mtable, and to mangle values with counts. (default: :const:`nil`).

**header**
	 A *list* specifying the augmented attributes names (and their order) used by default for the header when writing the mtable to files. Augmented meaning that the *list* is concatenated to the *list* held by the parent mtable during initialization. 
	 (default: :literal:`{'name', 'type', 'title', 'origin', 'date', 'time', 'refcol'}`).

**column**
	 A *list* specifying the augmented columns names (and their order) used by default for the columns when writing the mtable to files. Augmented meaning that the *list* is concatenated to the *list* held by the parent mtable during initialization. (default: :const:`nil`).

**novector**
	 A *logical* specifying to not convert (:literal:`novector == true`) columns containing only numbers to vectors during the insertion of the second row. The attribute can also be a *list* specifying the columns names to remove from the specialization. If the *list* is empty or :literal:`novector ~= true`, all numeric columns will be converted to vectors, and support all methods and operations from the :doc:`linear algebra <mad_mod_linalg>` module. (default: :const:`nil`).

**owner**
	 A *logical* specifying if an *empty* mtable is a view with no data (:literal:`owner ~= true`), or a mtable holding data (:literal:`owner == true`). (default: :const:`nil`).

**reserve**
	 A *number* specifying an estimate of the maximum number of rows stored in the mtable. If the value is underestimated, the mtable will still expand on need. (default: :literal:`8`).


**Warning**: the following private and read-only attributes are present in all mtables and should *never be used, set or changed*; breaking this rule would lead to an *undefined behavior*:

**__dat**
	 A *table* containing all the private data of mtables.

**__seq**
	 A *sequence* attached to the mtable by the :literal:`survey` and :var:`track` commands and used by the methods receiving a *reference* to an element as argument. (default: :const:`nil`).

**__cycle**
	 A *reference* to the row registered with the :literal:`:cycle` method. (default: :const:`nil`).


Methods
-------

The :literal:`mtable` object provides the following methods:

**nrow**
	 A *method*	:literal:`()` returning the *number* of rows in the mtable.

**ncol**
	 A *method*	:literal:`()` returning the *number* of columns in the mtable.

**ngen**
	 A *method*	:literal:`()` returning the *number* of columns generators in the mtable. The *number* of columns with data is given by :literal:`:ncol() - :ngen()`.

**colname**
	 A *method*	:literal:`(idx)` returning the *string* name of the :literal:`idx`-th column in the mtable or :const:`nil`.

**colnames**
	 A *method*	:literal:`([lst])` returning the *list* :literal:`lst` (default: :literal:`{}`) filled with all the columns names of the mtable.

**index**
	 A *method*	:literal:`(idx)` returning a positive index, or :const:`nil`. If :literal:`idx` is negative, it is reflected versus the size of the mtable, e.g. :literal:`-1` becomes :literal:`#self`, the index of the last row.

**name_of**
	 A *method*	:literal:`(idx, [ref])` returning a *string* corresponding to the (mangled) *value* from the reference column of the row at the index :literal:`idx`, or :const:`nil`. A row *value* appearing more than once in the reference column will be mangled with an absolute count, e.g. :literal:`mq[3]`, or a relative count versus the reference row determined by :literal:`:index_of(ref)`, e.g. :literal:`mq{-2}`.

**index_of**
	 A *method*	:literal:`(a, [ref], [dir])` returning a *number* corresponding to the positive index of the row determined by the first argument or :const:`nil`. If :literal:`a` is a *number* (or a *string* representing a *number*), it is interpreted as the index of the row and returned as a second *number*. If :literal:`a` is a *string*, it is interpreted as the (mangled) *value* of the row in the reference column as returned by :literal:`:name_of`. Finally, :literal:`a` can be a *reference* to an element to search for **if** the mtable has both, an attached sequence, and a column named :literal:`'eidx'` mapping the indexes of the elements to the attached sequence. [#f1]_ The argument :literal:`ref` (default: :const:`nil`) specifies the reference row determined by :literal:`:index_of(ref)` to use for relative indexes, for decoding mangled values with relative counts, or as the reference row to start searching from. The argument :literal:`dir` (default: :const:`1`) specifies the direction of the search with values :const:`1` (forward), :literal:`-1` (backward), or :const:`0` (no direction), which correspond respectively to the rounding methods :literal:`ceil`, :literal:`floor` and :literal:`round` from the lua math module.

**range_of**
	 A *method*	:literal:`([rng], [ref], [dir])` returning three *number*\ s corresponding to the positive indexes *start* and *end* of the range and its direction *dir* (default: :const:`1`), or :const:`nil` for an empty range. If :literal:`rng` is omitted, it returns :const:`1`, :literal:`#self`, :const:`1`, or :literal:`#self`, :const:`1`, :literal:`-1` if :literal:`dir` is negative. If :literal:`rng` is a *number* or a *string* with no :literal:`'/'` separator, it is interpreted as *start* and *end*, both determined by :literal:`:index_of`. If :literal:`rng` is a *string* containing the separator :literal:`'/'`, it is split in two *string*\ s interpreted as *start* and *end*, both determined by :literal:`:index_of`. If :literal:`rng` is a *list*, it will be interpreted as { *start*, *end*, :literal:`[ref]`, :literal:`[dir]` }, both determined by :literal:`:index_of`. The arguments :literal:`ref` and :literal:`dir` are forwarded to all invocations of :literal:`:index_of` with a higher precedence than ones in the *list* :literal:`rng`, and a runtime error is raised if the method returns :const:`nil`, i.e. to disambiguate between a valid empty range and an invalid range.

**length_of**
	 A *method*	:literal:`([rng], [ntrn], [dir])` returning a *number* specifying the length of the range optionally including :literal:`ntrn` extra turns (default: :const:`0`), and calculated from the indexes returned by :literal:`:range_of([rng], nil, [dir])`.

**get**
	 A *method*	:literal:`(row, col, [cnt])` returning the *value* stored in the mtable at the cell :literal:`(row,col)`, or :const:`nil`. If :literal:`row` is a not a row index determined by :literal:`:index(row)`, it is interpreted as a (mangled) *value* to search in the reference column, taking into account the count :literal:`cnt` (default: :const:`1`). If :literal:`col` is not a column index, it is interpreted as a column name.

**set**
	 A *method*	:literal:`(row, col, val, [cnt])` returning the mtable itself after updating the cell :literal:`(row,col)` to the value :var:`val`, or raising an error if the cell does not exist. If :literal:`row` is a not a row index determined by :literal:`:index(row)`, it is interpreted as a (mangled) *value* to search in the reference column, taking into account the count :literal:`cnt` (default: :const:`1`). If :literal:`col` is not a column index, it is interpreted as a column name.

**getcol**
	 A *method*	:literal:`(col)` returning the column :literal:`col`, or :const:`nil`. If :literal:`col` is not a column index, it is interpreted as a column name.

**setcol**
	 A *method*	:literal:`(col, val)` returning the mtable itself after updating the column :literal:`col` with the values of :var:`val`, or raising an error if the column does not exist. If :literal:`col` is not a column index, it is interpreted as a column name. If the column is a generator, so must be :var:`val` or an error will be raised. If the column is not a generator and :var:`val` is a *callable* :literal:`(ri)`, it will be invoked with the row index :literal:`ri` as its sole argument, using its returned value to update the column cell. Otherwise :var:`val` must be an *iterable* or an error will be raised. If the column is already a specialized *vector*, the *iterable* must provide enough numbers to fill it entirely as :const:`nil` is not a valid value.

**inscol**
	 A *method*	:literal:`([ref], col, val, [nvec])` returning the mtable itself after inserting the column data :var:`val` with the *string* name :literal:`col` at index :literal:`ref` (default: :literal:`:ncol()+1`). If :literal:`ref` is not a column index, it is interpreted as a column name. If :var:`val` is a *callable* :literal:`(ri)`, it will be added as a column generator. Otherwise :var:`val` must be an *iterable* or an error will be raised. The *iterable* will used to fill the new column that will be specialized to a *vector* if its first value is a *number* and :literal:`nvec ~= true` (default: :const:`nil`).

**addcol**
	 A *method*	:literal:`(col, val, [nvec])` equivalent to :literal:`:inscol(nil, col, val, [nvec])`.

**remcol**
	 A *method*	:literal:`(col)` returning the mtable itself after removing the column :literal:`col`, or raising an error if the column does not exist. If :literal:`col` is not a column index, it is interpreted as a column name.

**rencol**
	 A *method*	:literal:`(col, new)` returning the mtable itself after renaming the column :literal:`col` to the *string* :literal:`new`, or raising an error if the column does not exist. If :literal:`col` is not a column index, it is interpreted as a column name.

**getrow**
	 A *method*	:literal:`(row, [ref])` returning the *mappable* (proxy) of the row determined by the method :literal:`:index_of(row, [ref])`, or :const:`nil`.

**setrow**
	 A *method*	:literal:`(row, val, [ref])` returning the mtable itself after updating the row at index determined by :literal:`:index_of(row, [ref])` using the values provided by the *mappable* :var:`val`, which can be a *list* iterated as pairs of (*index*, *value*) or a *set* iterated as pairs of (*key*, *value*) with *key* being the column names, or a combination of the two. An error is raised if the column does not exist.

**insrow**
	 A *method*	:literal:`(row, val, [ref])` returning the mtable itself after inserting a new row at index determined by :literal:`:index_of(row, [ref])` and filled with the values provided by the *mappable* :var:`val`, which can be a *list* iterated as pairs of (*index*, *value*) or a *set* iterated as pairs of (*key*, *value*) with *key* being the column names or a combination of the two.

**addrow**
	 A *method*	:literal:`(val)` equivalent to :literal:`:insrow(#self+1, val)`.

**remrow**
	 A *method*	:literal:`(row, [ref])` returning the mtable itself after removing the row determined by the method :literal:`:index_of(row, [ref])`, or raising an error if the row does not exist.

**swprow**
	 A *method*	:literal:`(row1, row2, [ref1], [ref2])` returning the mtable itself after swapping the content of the rows, both determined by the method :literal:`:index_of(row, [ref])`, or raising an error if one of the row does not exist.

**clrrow**
	 A *method*	:literal:`(row, [ref])` returning the mtable itself after clearing the row determined by the method :literal:`:index_of(row, [ref])`, or raising an error if the row does not exist; where clearing the row means to set *vector* value to :const:`0` and :const:`nil` otherwise.

**clear**
	 A *method*	:literal:`()` returning the mtable itself after clearing all the rows, i.e. :literal:`#self == 0`, with an opportunity for new columns specialization.

**iter**
	 A *method*	:literal:`([rng], [ntrn], [dir])` returning an iterator over the mtable rows. The optional range is determined by :literal:`:range_of([rng], [dir])`, optionally including :literal:`ntrn` turns (default: :const:`0`). The optional direction :literal:`dir` specifies the forward :const:`1` or the backward :literal:`-1` direction of the iterator. If :literal:`rng` is not provided and the mtable is cycled, the *start* and *end* indexes are determined by :literal:`:index_of(self.__cycle)`. When used with a generic :literal:`for` loop, the iterator returns at each rows the index and the row *mappable* (proxy).

**foreach**
	 A *method*	:literal:`(act, [rng], [sel], [not])` returning the mtable itself after applying the action :literal:`act` on the selected rows. If :literal:`act` is a *set* representing the arguments in the packed form, the missing arguments will be extracted from the attributes :literal:`action`, :literal:`range`, :literal:`select` and :literal:`default`. The action :literal:`act` must be a *callable* :literal:`(row, idx)` applied to a row passed as first argument and its index as second argument. The optional range is used to generate the loop iterator :literal:`:iter([rng])`. The optional selector :literal:`sel` is a *callable* :literal:`(row, idx)` predicate selecting eligible rows for the action from the row itself passed as first argument and its index as second argument. The selector :literal:`sel` can be specified in other ways, see :doc:`row selections <mad_mod_numrange>` for details. The optional *logical* :literal:`not` (default: :const:`false`) indicates how to interpret default selection, as *all* or *none*, depending on the semantic of the action. [#f2]_

**select**
	 A *method*	:literal:`([rng], [sel], [not])` returning the mtable itself after selecting the rows using :literal:`:foreach(sel_act, [rng], [sel], [not])`. By default mtable have all their rows deselected, the selection being stored as *boolean* in the column at index :const:`0` and named :func:`is_selected`.

**deselect**
	 A *method*	:literal:`([rng], [sel], [not])` returning the mtable itself after deselecting the rows using :literal:`:foreach(desel_act, [rng], [sel], [not])`. By default mtable have all their rows deselected, the selection being stored as *boolean* in the column at index :const:`0` and named :func:`is_selected`.

**filter**
	 A *method*	:literal:`([rng], [sel], [not])` returning a *list* containing the positive indexes of the rows determined by :literal:`:foreach(filt_act, [rng], [sel], [not])`, and its size.

**insert**
	 A *method*	:literal:`(row, [rng], [sel])` returning the mtable itself after inserting the rows in the *list* :literal:`row` at the indexes determined by :literal:`:filter([rng], [sel], true)`. If the arguments are passed in the packed form, the extra attribute :literal:`rows` will be used as a replacement for the argument :literal:`row`, and if the attribute :literal:`where="after"` is defined then the rows will be inserted after the selected indexes. The insertion scheme depends on the number :math:`R` of rows in the *list* :literal:`row` versus the number :math:`S` of rows selected by :literal:`:filter`; :math:`1\times 1` (one row inserted at one index), :math:`R\times 1` (:math:`R` rows inserted at one index), :math:`1\times S` (one row inserted at :math:`S` indexes) and :math:`R\times S` (:math:`R` rows inserted at :math:`S` indexes). Hence, the insertion schemes insert respectively :math:`1`, :math:`R`, :math:`S`, and :math:`\min(R, S)` rows.

**remove**
	 A *method*	:literal:`([rng], [sel])` returning the mtable itself after removing the rows determined by :literal:`:filter([rng], [sel], true)`.

**sort**
	 A *method*	:literal:`(cmp, [rng], [sel])` returning the mtable itself after sorting the rows at the indexes determined by :literal:`:filter([rng], [sel], true)` using the ordering *callable* :literal:`cmp(row1, row2)`. The arguments :literal:`row1` and :literal:`row2` are *mappable* (proxies) referring to the current rows being compared and providing access to the columns values for the comparison. [#f3]_ The argument :literal:`cmp` can be specified in a compact ordering form as a *string* that will be converted to an ordering *callable* by the function :literal:`str2cmp` from the :doc:`utility <mad_mod_numrange>` module. For example, the *string* "-y,x" will be converted by the method to the following *lambda* :literal:`\\r1,r2 -> r1.y > r2.y or r1.y == r2.y and r1.x < r2.x`, where :literal:`y` and :literal:`x` are the columns used to sort the mtable in descending (`-`) and ascending (:literal:`+`) order respectively. The compact ordering form is not limited in the number of columns and avoids making mistakes in the comparison logic when many columns are involved.

**cycle**
	 A *method*	:literal:`(a)` returning the mtable itself after checking that :literal:`a` is a valid reference using :literal:`:index_of(a)`, and storing it in the :literal:`__cycle` attribute, itself erased by the methods editing the mtable like :literal:`:insert`, :literal:`:remove` or :literal:`:sort`.

**copy**
	 A *method*	:literal:`([name], [owner])` returning a new mtable from a copy of :literal:`self`, with the optional :literal:`name` and the optional attribute :literal:`owner` set. If the mtable is a view, so will be the copy unless :literal:`owner == true`.

**is_view**
	 A *method*	:literal:`()` returning :const:`true` if the mtable is a view over another mtable data, :const:`false` otherwise.

**set_readonly**
	 Set the mtable as read-only, including the columns and the rows proxies.

**read**
	 A *method*	:literal:`([filname])` returning a new instance of :literal:`self` filled with the data read from the file determined by :literal:`openfile(filename, 'r', {'.tfs','.txt','.dat'})` from the :doc:`utility <mad_mod_miscfuns>` module. This method can read columns containing the data types *nil*, *boolean*, *number*, *complex number*, (numerical) *range*, and (quoted) *string*. The header can also contain tables saved as *string* and decoded with *function* :literal:`str2tbl` from the :doc:`utility <mad_mod_miscfuns>` module.

**write**
	 A *method*	:literal:`([filname], [clst], [hlst], [rsel])` returning the mtable itself after writing its content to the file determined by :literal:`openfile(filename, 'w', {'.tfs', '.txt', '.dat'})` from the :doc:`utility <mad_mod_miscfuns>` module. The columns to write and their order is determined by :literal:`clst` or :literal:`self.column` (default: :const:`nil` :math:`\equiv` all columns). The attributes to write in the header and their order is determined by :literal:`hlst` or :literal:`self.header`. The *logical* :literal:`rsel` indicates to save all rows or only rows selected by the :literal:`:select` method (:literal:`rsel == true`). This method can write columns containing the data types *nil*, *boolean*, *number*, *complex number*, (numerical) *range*, and (quoted) *string*. The header can also contain tables saved as *string* and encoded with *function* :literal:`tbl2str` from the :doc:`utility <mad_mod_miscfuns>` module.

**print**
	 A *method*	:literal:`([clst], [hlst], [rsel])` equivalent to :literal:`:write(nil, [clst], [hlst], [rsel])`.

**save_sel**
	 A *method*	:literal:`([sel])` saving the rows selection to the optional *iterable* :literal:`sel` (default: :literal:`{}`) and return it.

**restore_sel**
	 A *method*	:literal:`(sel)` restoring the rows selection from the *iterable* :literal:`sel`. The indexes of :literal:`sel` must match the indexes of the rows in the mtable.

**make_dict**
	 A *method*	:literal:`([col])` returning the mtable itself after building the rows dictionnary from the values of the reference column determined by :literal:`col` (default: :literal:`refcol`) for fast row access. If :literal:`col` is not a column index, it is interpreted as a column name except for the special name :literal:`'none'` that disables the rows dictionnary and reset :literal:`refcol` to :const:`nil`.

**check_mtbl**
	 A *method*	:literal:`()` checking the integrity of the mtable and its dictionary (if any), for debugging purpose only.


Metamethods
-----------

The :literal:`mtable` object provides the following metamethods:


**__len**
	 A *metamethod*	:literal:`()` called by the length operator :literal:`#` to return the number of rows in the mtable.

**__add**
	 A *metamethod*	:literal:`(val)` called by the plus operator :literal:`+` returning the mtable itself after appending the row :var:`val` at its end, similiar to the :literal:`:addrow` method.

**__index**
	 A *metamethod*	:literal:`(key)` called by the indexing operator :literal:`[key]` to return the *value* of an attribute determined by *key*. The *key* is interpreted differently depending on its type with the following precedence:

		#. A *number* is interpreted as a row index and returns an *iterable* on the row (proxy) or :const:`nil`.
		#. Other *key* types are interpreted as *object* attributes subject to object model lookup.
		#. If the *value* associated with *key* is :const:`nil`, then *key* is interpreted as a column name and returns the column if it exists, otherwise...
		#. If *key* is not a column name, then *key* is interpreted as a value in the reference column and returns either an *iterable* on the row (proxy) determined by this value or an *iterable* on the rows (proxies) holding this non-unique value. [#f4]_
		#. Otherwise returns :const:`nil`.

**__newindex**
	A *metamethod*	:literal:`(key, val)` called by the assignment operator :literal:`[key]=val` to create new attributes for the pairs (*key*, *value*). If *key* is a *number* or a value specifying a row in the reference column or a *string* specifying a column name, the following error is raised:

	.. code-block::
		
		"invalid mtable write access (use 'set' methods)"


**__init**
	 A *metamethod*	:literal:`()` called by the constructor to build the mtable from the column names stored in its *list* part and some attributes, like :literal:`owner`, :literal:`reserve` and :literal:`novector`.

**__copy**
	 A *metamethod*	:literal:`()` similar to the *method* :literal:`copy`.

The following attribute is stored with metamethods in the metatable, but has different purpose:

**__mtbl**
	 A unique private *reference* that characterizes mtables.


MTables creation
----------------
.. _sec.tbl.create:

During its creation as an *object*, an mtable can defined its attributes as any object, and the *list* of its column names, which will be cleared after its initialization. Any column name in the *list* that is enclosed by braces is designated as the refererence column for the dictionnary that provides fast row indexing, and the attribute :literal:`refcol` is set accordingly.

Some attributes are considered during the creation by the *metamethod* :literal:`__init`, like :literal:`owner`, :literal:`reserve` and :literal:`novector`, and some others are initialized with defined values like :literal:`type`, :literal:`title`, :literal:`origin`, :literal:`date`, :literal:`time`, and :literal:`refcol`. The attributes :literal:`header` and :literal:`column` are concatenated with the the parent ones to build incrementing *list* of attributes names and columns names used by default when writing the mtable to files, and these lists are not provided as arguments.

The following example shows how to create a mtable form a *list* of column names add rows:

.. code-block::
	
	local mtable in MAD
	local tbl = mtable 'mytable' {
	
	   {'name'}, 'x', 'y' } -- column 'name' is the refcol
	  + { 'p11', 1.1, 1.2 }
	  + { 'p12', 2.1, 2.2 }
	  + { 'p13', 2.1, 3.2 }
	  + { 'p11', 3.1, 4.2 }
	print(tbl.name, tbl.refcol, tbl:getcol'name')
	-- display: mytable  name   mtable reference column: 0x010154cd10

**Pitfall:** When a column is named :literal:`'name'`, it must be explicitly accessed, e.g. with the :literal:`:getcol` method, as the indexing operator :literal:`[]` gives the precedence to object's attributes and methods. Hence, :literal:`tbl.name` returns the table name :literal:`'mytable'`, not the column :literal:`'name'`.

Rows selections
---------------
.. _sec.tbl.rowsel:

The row selection in mtable use predicates in combination with iterators. The mtable iterator manages the range of rows where to apply the selection, while the predicate says if a row in this range is illegible for the selection. In order to ease the use of methods based on the :literal:`:foreach` method, the selector predicate :literal:`sel` can be built from different types of information provided in a *set* with the following attributes:

**selected**
	 A *boolean* compared to the rows selection stored in column :literal:`'is_selected'`.

**pattern**
	 A *string* interpreted as a pattern to match the *string* in the reference column, which must exist, using :literal:`string.match` from the standard library, see `Lua 5.2 <http://github.com/MethodicalAcceleratorDesign/MADdocs/blob/master/lua52-refman-madng.pdf>`_ §6.4 for details. If the reference column does not exist, it can be built using the :meth:`make_dict` method.

**list**
	 An *iterable* interpreted as a *list* used to build a *set* and select the rows by their name, i.e. the built predicate will use :literal:`tbl[row.name]` as a *logical*, meaning that column :literal:`name` must exists. An alternate column name can be provided through the key :literal:`colname`, i.e. used as :literal:`tbl[row[colname]]`. If the *iterable* is a single item, e.g. a *string*, it will be converted first to a *list*.

**table**
	 A *mappable* interpreted as a *set* used to select the rows by their name, i.e. the built predicate will use :literal:`tbl[row.name]` as a *logical*, meaning that column :literal:`name` must exists. If the *mappable* contains a *list* or is a single item, it will be converted first to a *list* and its *set* part will be discarded.

**kind**
	 An *iterable* interpreted as a *list* used to build a *set* and select the rows by their kind, i.e. the built predicate will use :literal:`tbl[row.kind]` as a *logical*, meaning that column :literal:`kind` must exists. If the *iterable* is a single item, e.g. a *string*, it will be converted first to a *list*. This case is equivalent to :literal:`list` with :literal:`colname='kind'`.

**select**
	 A *callable* interpreted as the selector itself, which allows to build any kind of predicate or to complete the restrictions already built above.

All these attributes are used in the aforementioned order to incrementally build predicates that are combined with logical conjunctions, i.e. :literal:`and`'ed, to give the final predicate used by the :literal:`:foreach` method. If only one of these attributes is needed, it is possible to pass it directly in :literal:`sel`, not as an attribute in a *set*, and its type will be used to determine the kind of predicate to build. For example, :literal:`tbl:foreach(act, "^MB")` is equivalent to :literal:`tbl:foreach{action=act, pattern="^MB"}`.

Indexes, names and counts
-------------------------

Indexing a mtable triggers a complex look up mechanism where the arguments will be interpreted in various ways as described in the metamethod :literal:`__index`. A *number* will be interpreted as a relative row index in the list of rows, and a negative index will be considered as relative to the end of the mtable, i.e. :literal:`-1` is the last row. Non-*number* will be interpreted first as an object key (can be anything), looking for mtable methods or attributes; then as a column name or as a row *value* in the reference column if nothing was found.

If a row exists but its *value* is not unique in the reference column, an *iterable* is returned. An *iterable* supports the length :literal:`#` operator to retrieve the number of rows with the same *value*, the indexing operator :literal:`[]` waiting for a count :math:`n` to retrieve the :math:`n`-th row from the start with that *value*, and the iterator :literal:`ipairs` to use with generic :literal:`for` loops.

The returned *iterable* is in practice a proxy, i.e. a fake intermediate object that emulates the expected behavior, and any attempt to access the proxy in another manner should raise a runtime error.

**Note:** Compared to the sequence, the indexing operator :literal:`[]` and the method :literal:`:index_of` of the mtable always interprets a *number* as a (relative) row index. To find a row from a :math:`s`-position [m] in the mtable if the column exists, use the functions :literal:`lsearch` or :literal:`bsearch` (if they are monotonic) from the :doc:`utility <mad_mod_miscfuns>` module.

The following example shows how to access to the rows through indexing and the *iterable*:

.. code-block::
	
	local mtable in MAD
	local tbl = mtable { {'name'}, 'x', 'y' } -- column 'name' is the refcol
	                   + { 'p11', 1.1, 1.2 }
	                   + { 'p12', 2.1, 2.2 }
	                   + { 'p13', 2.1, 3.2 }
	                   + { 'p11', 3.1, 4.2 }
	print(tbl[ 1].y) -- display: 1.2
	print(tbl[-1].y) -- display: 4.2
	
	print(#tbl.p11, tbl.p12.y, tbl.p11[2].y)            -- display: 2 2.2 4.2
	for _,r in ipairs(tbl.p11) do io.write(r.x," ") end -- display: 1.1 3.1
	for _,v in ipairs(tbl.p12) do io.write(v,  " ") end -- display: 'p12' 2.1 2.2
	
	-- print name of point with name p11 in absolute and relative to p13.
	print(tbl:name_of(4))       -- display: p11[2]  (2nd p11 from start)
	print(tbl:name_of(1, -2))   -- display: p11{-1} (1st p11 before p13)

The last two lines of code display the name of the same row but mangled with absolute and relative counts.

Iterators and ranges
--------------------

Ranging a mtable triggers a complex look up mechanism where the arguments will be interpreted in various ways as described in the method :literal:`:range_of`, itself based on the methods :literal:`:index_of` and :literal:`:index`. The number of rows selected by a mtable range can be computed by the :literal:`:length_of` method, which accepts an extra *number* of turns to consider in the calculation.

The mtable iterators are created by the method :literal:`:iter`, based on the method :literal:`:range_of` as mentioned in its description and includes an extra *number* of turns as for the method :literal:`:length_of`, and a direction :const:`1` (forward) or :literal:`-1` (backward) for the iteration.

The method :literal:`:foreach` uses the iterator returned by :literal:`:iter` with a range as its sole argument to loop over the rows where to apply the predicate before executing the action. The methods :literal:`:select`, :literal:`:deselect`, :literal:`:filter`, :literal:`:insert`, and :literal:`:remove` are all based directly or indirectly on the :literal:`:foreach` method. Hence, to iterate backward over a mtable range, these methods have to use either its *list* form or a numerical range. For example the invocation :literal:`tbl:foreach(\\r -> print(r.name), {-2, 2, nil, -1})` will iterate backward over the entire mtable excluding the first and last rows, equivalently to the invocation :literal:`tbl:foreach(\\r -> print(r.name), -2..2..-1)`.

The following example shows how to access to the rows with the :literal:`:foreach` method:

.. code-block::
	
	local mtable in MAD
	local tbl = mtable { {'name'}, 'x', 'y' }
	                   + { 'p11', 1.1, 1.2 }
	                   + { 'p12', 2.1, 2.2 }
	                   + { 'p13', 2.1, 3.2 }
	                   + { 'p11', 3.1, 4.2 }
	
	local act = \r -> print(r.name, r.y)
	tbl:foreach(act, -2..2..-1)
	-- display:  p13   3.2
	!            p12   2.2
	tbl:foreach(act, "p11[1]/p11[2]")
	-- display:  p11   1.2
	!            p12   2.2
	!            p13   3.2
	!            p11   4.2
	tbl:foreach{action=act, range="p11[1]/p13"}
	-- display:  p11   1.2
	!            p12   2.2
	!            p13   3.2
	tbl:foreach{action=act, pattern="[^1]$"}
	-- display:  p12   2.2
	!            p13   3.2
	local act = \r -> print(r.name, r.y, r.is_selected)
	tbl:select{pattern="p.1"}:foreach{action=act, range="1/-1"}
	-- display:  p11   1.2   true
	!            p12   2.2   nil
	!            p13   3.2   nil
	!            p11   4.2   true


Examples
--------

Creating a MTable
"""""""""""""""""

The following example shows how the :var:`track` command, i.e. :literal:`self` hereafter, creates its MTable:

.. code-block::
	
	local header = { -- extra attributes to save in track headers
	  'direction', 'observe', 'implicit', 'misalign', 'deltap', 'lost' }
	
	local function make_mtable (self, range, nosave)
	  local title, dir, observe, implicit, misalign, deltap, savemap in self
	  local sequ, nrow = self.sequence, nosave and 0 or 16
	
	  return mtable(sequ.name, { -- keep column order!
	    type='track', title=title, header=header,
	    direction=dir, observe=observe, implicit=implicit, misalign=misalign,
	    deltap=deltap, lost=0, range=range, reserve=nrow, __seq=sequ,
	    {'name'}, 'kind', 's', 'l', 'id', 'x', 'px', 'y', 'py', 't', 'pt',
	    'slc', 'turn', 'tdir', 'eidx', 'status', savemap and '__map' or nil })
	end


Extending a MTable
""""""""""""""""""

The following example shows how to extend the MTable created by a :var:`twiss` command with the elements tilt, angle and integrated strengths from the attached sequence:

.. code-block::
	
	-- The prelude creating the sequence seq is omitted.
	local tws = twiss { sequence=seq, method=4, cofind=true }
	
	local is_integer in MAD.typeid
	tws:addcol('angle', \ri => -- add angle column
	      local idx = tws[ri].eidx
	      return is_integer(idx) and tws.__seq[idx].angle or 0 end)
	   :addcol('tilt', \ri => -- add tilt column
	      local idx = tws[ri].eidx
	      return is_integer(idx) and tws.__seq[idx].tilt or 0 end)
	
	for i=1,6 do -- add kil and kisl columns
	tws:addcol('k'..i-1..'l', \ri =>
	      local idx = tws[ri].eidx
	      if not is_integer(idx) then return 0 end -- implicit drift
	      local elm = tws.__seq[idx]
	      return (elm['k'..i-1] or 0)*elm.l + ((elm.knl or {})[i] or 0)
	    end)
	   :addcol('k'..i-1..'sl', \ri =>
	      local idx = tws[ri].eidx
	      if not is_integer(idx) then return 0 end -- implicit drift
	      local elm = tws.__seq[idx]
	      return (elm['k'..i-1..'s'] or 0)*elm.l + ((elm.ksl or {})[i] or 0)
	    end)
	end
	
	local cols = {'name', 'kind', 's', 'l', 'angle', 'tilt',
	    'x', 'px', 'y', 'py', 't', 'pt',
	    'beta11', 'beta22', 'alfa11', 'alfa22', 'mu1', 'mu2', 'dx', 'ddx',
	    'k1l', 'k2l', 'k3l', 'k4l', 'k1sl', 'k2sl', 'k3sl', 'k4sl'}
	
	tws:write("twiss", cols) -- write header and columns to file twiss.tfs

Hopefully, the :doc:`physics <mad_mod_gphys>` module provides the *function* :literal:`melmcol(mtbl, cols)` to achieve the same task easily:

.. code-block::
	
	-- The prelude creating the sequence seq is omitted.
	local tws = twiss { sequence=seq, method=4, cofind=true }
	
	-- Add element properties as columns
	local melmcol in MAD.gphys
	local melmcol(tws, {'angle', 'tilt', 'k1l' , 'k2l' , 'k3l' , 'k4l',
	                                     'k1sl', 'k2sl', 'k3sl', 'k4sl'})
	
	-- write TFS table
	tws:write("twiss", {
	    'name', 'kind', 's', 'l', 'angle', 'tilt',
	    'x', 'px', 'y', 'py', 't', 'pt',
	    'beta11', 'beta22', 'alfa11', 'alfa22', 'mu1', 'mu2', 'dx', 'ddx',
	    'k1l', 'k2l', 'k3l', 'k4l', 'k1sl', 'k2sl', 'k3sl', 'k4sl'})


.. rubric:: Footnotes

.. [#f1] These information are usually provided by the command creating the :literal:`mtable`, like :literal:`survey` and :var:`track`.
.. [#f2] For example, the :literal:`:remove` method needs :literal:`not=true` to *not* remove all rows if no selector is provided.
.. [#f3] A *mappable* supports the length operator :literal:`#`, the indexing operator :literal:`[]`, and generic :literal:`for` loops with :literal:`pairs`.
.. [#f4] An *iterable* supports the length operator :literal:`#`, the indexing operator :literal:`[]`, and generic :literal:`for` loops with :literal:`ipairs`.
