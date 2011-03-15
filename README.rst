======
nestly
======

Nestly is a collection of functions designed to make running software with combinatorial choices of parameters easier.

.. However, importantly, the set of parameters is fixed for all of the simulations.

The vision here is that we take a fixed set of parameters and generate a single type of output for each defined combination, which can then be combined in some way for comparison and retrieval.
We would like to set things up tidily with nested directories for output reflecting nested parameter choices.

Note that a fixed set of parameters is not the only way to go, and in Extensions_ below we describe a more general setup.

nestly operates in a few steps (see the ``examples/`` directory):

1. You define the argument combinations (see the ``examples/`` directory) to be used in the runs.
   This usually consists of building an OrderedDict of parameter choices, then passing it to ``nestly.nestly.build``
2. Nestly creates a (nested) directory structure for the output, where leaf nodes have JSON files detailing parameters chosen.
3. You define a command or script to be run for each parameter combination, with variables to substitute parameter choices at that node.
4. ``nestrun`` substitutes the values from each control file to create the commands to be run and, optionally, runs them.

Parameter lists
===============
In the main control dictionary, we supply a function which takes the control so far and spits out a list of the next things to use.

Sample functions for working with file globs are defined in ``nestly/nestly.py``. You're also free to define your own:
Any function accepting a single argument (the control dictionary), returning an iterable of ``(name, value)`` tuples. 
The name provided becomes the name the nested directory for the parameter choice; the value is what's actually written 
to the JSON file.

the commonality is that each one makes a directory returns a list of choices for the next param
the function takes those choices, chdirs, updates the control dictionary, then recurs

Example
-------

Mirroring the code in ``examples/basic_nest/make_nest.py``: imagine you want run a command with all combinations of the following parameters:

========== ==============================
Option     Choices
---------- ------------------------------
strategy   approximate, exhaustive
---------- ------------------------------
run_count  10, 100, 1000
---------- ------------------------------
input file any file matching inputs/file*
========== ==============================

Running ``make_nest.py``, you get a directory tree like::

  ├── approximate
  │   ├── 10
  │   │   ├── file1
  │   │   │   └── control.json
  │   │   ├── file2
  │   │       └── control.json
  │   ├── 100
  │   │   ├── file1
  │   │   │   └── control.json
  │   │   ├── file2
  │   │       └── control.json
  │   └── 1000
  │       ├── file1
  │       │   └── control.json
  │       ├── file2
  │           └── control.json
  └── exhaustive
      ├── 10
      │   ├── file1
      │   │   └── control.json
      │   ├── file2
      │       └── control.json
      ├── 100
      │   ├── file1
      │   │   └── control.json
      │   ├── file2
      │       └── control.json
      └── 1000
          ├── file1
          │   └── control.json
          ├── file2
              └── control.json

With the final ``control.json`` reading::

  {
      "input_file": "/Users/cmccoy/Development/nestly/examples/basic_nest/inputs/file3", 
      "run_count": "1000", 
      "strategy": "exhaustive"
  }

The control files created then serve as inputs to ``nestrun`` for template substition, for example::

  nestrun --dryrun ---savecmdfile command.sh \
          --template='my_command -s $strategy --count=$run_count $input_file' \
          $(find runs -name "control.json")


.. Implementation
    --------------
    At each node we specify function giving action of param on the control file, and children 


    Results analysis
    ================
    Want to be able to extract results by keyword



Extensions
==========

Parameter trees
---------------
One natural extension of a list is a tree.

If there are some types of simulation which require different number of parameter choices.
for example, say we had a no rate var sim and a rate var sim.
would need to collapse all of those choices into a single one.

will make things complex from the database side of things-- rather than a parameter list we have combinations of parameters...
but we need a complete control param dictionary--
