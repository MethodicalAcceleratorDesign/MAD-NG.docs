Python
======

Within MAD, there is a low level communication process that allows transfer of data between MAD and python. Most of the work between sending and receiving data must be performed by the user, however this is a robust and flexible way to communicate between the two languages. For a higher level way of communication, you can use the `pymadng <https://pypi.org/project/pymadng/>`_ package on PyPI.

Preqrequisites
--------------

- Python 3.6 or higher
- NumPy 1.11.0 or higher
- A working binary of MAD-NG

Because the file "madp\_pymad.py" is a python module, it must be in the same directory as the python script that is importing it, or you must add the directory containing "madp\_pymad.py" to your path by using the following code:

.. code-block:: python
    :caption: Adding the directory containing "madp\_pymad.py" to your path

    import sys
    sys.path.append("path/to/module") # where "path/to/module/madp_pymad.py" is located

Now you can import the module, I recommend using the following code to import the module:

.. code-block:: python
    :caption: Importing the module

    from madp_pymad import mad_process as MAD

Finally, to start the mad process from python, you must tell python where your MAD-NG binary is located:

.. code-block:: python
    :caption: Starting the MAD-NG process

    mad = MAD("path/to/mad") # where "path/to/mad" is the location of your MAD-NG binary

Communication Protocol
----------------------

The communication protocol has been presented previously in a meeting that can be found here: `MAD-NG Python interface <https://indico.cern.ch/event/1224204/>`_. The essential points are:

- PyMAD-NG communicates through pipes (first in, first out)
- Communication occurs by sending MAD-NG scripts (as strings) to MAD
- Retrieve data from MAD to Python pipe.
- The stdout of MAD is redirected to the stdout of Python (not intercepted by PyMAD-NG)

The first point is the most consequential for the user, as it means that the order in which you send data to MAD-NG is the order in which it will be received and vice versa for retrieving data. Therefore, you must adhere to the following rules:

.. important:: 
    - **Before you receive any data from MAD-NG, you must always ask MAD-NG to send the data.**
    - **Before you send data to MAD-NG, you must always send MAD-NG the instructions to read the data.**
  

.. code-block:: python
    :caption: An example of using the mad_process object to communicate with MAD-NG
    
    #Load the mad_process
    from madp_pymad import mad_process as MAD # Assuming that the madp_pymad.py file is in the current directory
    mad = MAD("./mad")                        # Assuming that the mad binary is in the current directory

    #Tell mad that it should expect data and then place it in 'a'
    mad.send("a = py:recv()")
    
    #Send the data
    mad.send(42)

    #Ask mad to send the data back
    mad.send("py:send(a)")

    #Read the data
    mad.recv() #-> 42


:meth:`mad.send() <pymadng.MAD.send>` and :meth:`mad.recv() <pymadng.MAD.recv>` are the main ways to communicate with MAD-NG and is extremely simple, for specific details on what data can be sent see the :class:`API Reference <pymadng.MAD>`.

For types that the equivalent of MAD-NG cannot be naturally found in numpy or python, you will be required to use a different function to *send* data (see below). To *receive* any data just use :meth:`mad.recv() <pymadng.MAD.recv>`.

.. _typestbl:

.. table:: Types that can be sent to MAD-NG and the function to use to send them
    
    +----------------------------------------+------------------------+----------------------------------------------+
    | Type in Python                         | Type in MAD            | Function to send from Python                 |
    +========================================+========================+==============================================+
    | None                                   | nil                    | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | str                                    | string                 | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | int                                    | number :math:`<2^{31}` | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | float                                  | number                 | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | complex                                | complex                | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | list                                   | table                  | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | bool                                   | bool                   | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | NumPy ndarray (dtype = np.float64)     | matrix                 | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | NumPy ndarray (dtype = np.complex128)  | cmatrix                | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | NumPy ndarray (dtype = np.int32)       | imatrix                | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | range                                  | irange                 | :meth:`send <pymadng.MAD.send>`              |
    +----------------------------------------+------------------------+----------------------------------------------+
    | start(float), stop(float), size(int)   | range                  | :meth:`send_rng <pymadng.MAD.send_rng>`      |
    +----------------------------------------+------------------------+----------------------------------------------+
    | start(float), stop(float), size(int)   | logrange               | :meth:`send_lrng <pymadng.MAD.send_lrng>`    |
    +----------------------------------------+------------------------+----------------------------------------------+
    || NumPy ndarray (dtype = np.uint8) and  || TPSA                  || :meth:`send_tpsa <pymadng.MAD.send_tpsa>`   |
    || NumPy ndarray (dtype = np.float64)    ||                       ||                                             |
    +----------------------------------------+------------------------+----------------------------------------------+
    || NumPy ndarray (dtype = np.uint8) and  || CTPSA                 || :meth:`send_ctpsa <pymadng.MAD.send_ctpsa>` |
    || NumPy ndarray (dtype = np.complex128) ||                       ||                                             |
    +----------------------------------------+------------------------+----------------------------------------------+

Since above does not cover all types that can be sent from MAD-NG to Python, there is a class within the module that can be used to receive higher levels of data, such as elements, mtables, sequences, etc... This class is called :class:`mad_ref` and is simply a wrapper that mimicks a MAD-NG object to allow you to retrieve attributes of the object through keys and indexing. This can occur continuously until you reach a primitive type (see above). In order to do this communication, python needs to know the name of this object in the MAD environment, which must be provided by the user when they send an object. For example:

.. code-block:: python

    #Load the mad_process
    from madp_pymad import mad_process as MAD
    mad = MAD("./mad")

    #Create an object of objects in MAD
    mad.send("""
    local object in MAD 

    obj = object {
        a = object {
            b = 42
        }
    }  
    py:send(obj)
    """)
    obj = mad.recv("obj") # -> mad_ref
    a = obj["a"]          # -> mad_ref
    b = a.b               # -> 42
    print(b)              # -> 42

If you give the wrong name for the object in the environment, you will create a reference to a (possibly) non-existent object. This will not cause an error, but will instead return a :class:`mad_ref` object that will return None for any attribute you try to access. In fact, as no data is actually passed from MAD to python, except for the fact an object was attempted to be sent, therefore you can do the following:

.. code-block:: python

    # Load MAD from pymadng
    from madp_pymad import mad_process as MAD
    mad = MAD("./mad")

    # Load mad_ref from pymadng
    from madp_pymad import mad_ref

    # Create an object of objects in MAD
    mad.send("""
    local object in MAD
    mad_obj = object {
        a = object {
            b = 42
        }
    }
    """)

    # Create a mad_ref object
    py_obj = mad_ref(mad, "mad_obj") # mad_ref needs the mad_process object (for communication) and the name of the object in MAD
    print(py_obj.a.b) # -> 42

    # Create a mad_ref object with the wrong name
    py_obj = mad_ref(mad, "wrong_name")
    print(py_obj.a) # -> Error

Receiving Variables
-------------------

You may be thinking that looking at above, it may become pretty arduous to have to always write :meth:`py:send(var)` and :meth:`mad.recv("var")` every time you want to send a variable from MAD to python. Therefore, there is a function that will do this for you, which is helpful, especially for the case of objects, where you don't want to unnecessarily use the variable name multiple times. This function is :meth:`recv_vars` and can be used as follows:

.. code-block:: python

    #Load the mad_process
    from madp_pymad import mad_process as MAD
    mad = MAD("./mad")

    #Create an object of objects in MAD
    mad.send("""
    local object in MAD 

    obj = object {
        a = object {
            b = 42
        }
    }  
    c = "Hello World"
    """)
    obj, c, a = mad.recv_vars("obj", "c", "obj.a") # -> mad_ref, str, mad_ref
    b = mad.recv_vars("obj.a.b")

    print(b == a.b, a.b == obj.a.b) # -> True, True
    print(c)                        # -> Hello World

Sending Variables
-----------------

Similarly, you may be thinking that looking at above, it may become pretty arduous to have to always write :meth:`py:recv()` and :meth:`mad.send(var)` every time you want to send a variable from python to MAD. Therefore, there is a function that will do this for you, which also is useful in the case of wanting to send multiple variables at once. This function is :meth:`send_vars` and can be used as follows:

.. code-block:: python

    #Load the mad_process
    from madp_pymad import mad_process as MAD
    mad = MAD("./mad")

    a, b, c, d = 42, 42.0, "Hello World", [1, 2, 3]
    mad.send_vars(a_m = a, b_m = b, c_m = c, d_m = d) # -> Sends a, b, c, d to MAD

    a2, b2, c2, d2 = mad.recv_vars("a_m", "b_m", "c_m", "d_m") # -> 42, 42.0, "Hello World", [1, 2, 3]

    print(a == a2, b == b2, c == c2, d == d2) # -> True, True, True, True

    #Equivalent to:
    mad.send("a_m2 = py:recv()").send(a)
    mad.send("b_m2 = py:recv()").send(b)
    mad.send("c_m2 = py:recv()").send(c)
    mad.send("d_m2 = py:recv()").send(d)
    mad.send("py:send(a_m2):send(b_m2):send(c_m2):send(d_m2)")

    print(
        a == mad.recv(), 
        b == mad.recv(), 
        c == mad.recv(), 
        d == mad.recv()
    ) # -> True, True, True, True


Error Handling
--------------

If an error occurs in MAD-NG, it will be printed to the stdout of the python process, however it has no effect on the python process and it will continue as if nothing happened. This could be problematic, such as if you do any of the following:

- Define a variable that needs to be used later in the script
  - If MAD-NG will throws an error before the variable definition, python will continue with future commands, which may not give the expected result.
- Attempt to send a variable from MAD to python
  - If MAD-NG throws an error before the variable is sent, python will attempt to receive a variable that does not exist, which will cause python to hang.

This is a short list that can be extended significantly, in other words, its not always ideal for python to not react to MAD erroring, therefore there are three functions at your disposal to handle errors:

- :meth:`mad.errhdlr()`
  
    .. code-block:: python

        mad.send("a = 42")
        mad.errhdlr(True)           # -> Turn on error handling
        mad.send("b = a/'a'")       # -> MAD has now errored (sends this down the pipe)
        print("Python not Errored") # -> This will still be printed
        mad.recv()                  # -> A RuntimeError will be raised due to the error in the pipe, stopping the python process
    
    .. code-block:: python

        mad.errhdlr(True)           # -> Turn on error handling
        mad.send("a = 42")
        mad.errhdlr(False)          # -> Turn off error handling
        mad.send("b = a/'a'")       # -> MAD has now errored (nothing is sent down the pipe)
        print("Python not Errored") # -> This will still be printed
        mad.send("py:send(b)")      # -> A None object will be sent down the pipe, as b does not exist
        print(mad.recv())           # -> None
        
- :meth:`mad.psend()` (Turns on error handling before sending and turns it off after sending)

    .. code-block:: python

        mad.send("a = 42")
        mad.psend("b = a/'a'")      # -> MAD has now errored (sends this down the pipe)
        print("Python not Errored") # -> This will still be printed
        mad.recv()                  # -> A RuntimeError will be raised due to the error in the pipe, stopping the python process

- :meth:`mad.precv()` (Turns on error handling before asking for the object and turns it off after receiving)

    .. code-block:: python

        mad.send("a = 42")
        print(mad.precv("a"))         # -> 42
        mad.send("b = a/'a'")         # -> MAD has now errored (nothing is sent down the pipe)

        mad.send("py:send(a)")        # -> Python continues as if nothing happened
        print(mad.recv())             # -> 42
        mad.precv("a.b.c")            # -> A RuntimeError will be raised as a.b is not indexable


Executing Python Code Sent from MAD
-----------------------------------

Finally there is a function that allows you to execute python code from MAD, this is :meth:`mad.recv_and_exec()`. This is potentially useful if you would like to use python to do some calculations and then send the result back to MAD automatically. In this function, you have the ability to add variables to the python environment, which can be used in the python code that is executed. By default, the only variables that are added to the python environment is the mad_process object, which is added as "mad" and the numpy library, which is added as "np". For example:

.. code-block:: python

     #Load the mad_process
    from madp_pymad import mad_process as MAD
    mad = MAD("./mad")

    mad.send("""
    a = py:send('mad.send(a)'):recv()
    mat = py:send('mad.send(np.array([${a}, ${a}, ${a}, ${a}], dtype=np.int32).reshape(2, 2))'%{a = a}):recv()
    mat:print()
    py:send([==[mad.send('''py:send([=[mad.send("py:send([[a = 100/2]])")]=])''')]==])
    """)

    mad.recv_and_exec({'a': 42})
    mad.recv_and_exec() # prints a 2x2matrix of 42
    mad.recv_and_exec()
    mad.recv_and_exec()
    a = mad.recv_and_exec()["a"]

    print(a) # -> 50