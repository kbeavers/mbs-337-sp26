YAML Reference
==============

This reference guide is designed to introduce you to YAML syntax. YAML (YAML
Ain't Markup Language) is basically JSON with a couple extra features, and meant
to be a little more human readable. We will be using YAML formatted configuration
files in the Docker compose sections later in the course, so it is important to become
familiar with the syntax.


YAML Basics
-----------

YAML syntax is similar to Python dictionaries, and we will usually see them as
key:value pairs. Values can include strings, numbers, booleans, null, lists,
and other dictionaries.

Previously, we saw a simple JSON object in dictionary form like:

.. code-block:: json

   {
     "key1": "value1",
     "key2": "value2"
   }

That same object in YAML looks like:

.. code-block:: yaml

   ---
   key1: value1
   key2: value2
   ...

Notice that YAML documents all start with three hyphens on the top line (``---``),
and end with an optional three dots (``...``) on the last line. Key:value pairs
are separated by colons, but consecutive key:value pairs are NOT separated by
commas.

We also mentioned that JSON supports list-like structures. YAML does too. So the
following valid JSON block:

.. code-block:: json

   [
     "thing1", "thing2", "thing3"
   ]

Appears like this in YAML:

.. code-block:: yaml

   ---
   - thing1
   - thing2
   - thing3
   ...

Elements of the same list all appear with a hyphen ``-`` at the same indent
level.

We previously saw this complex data structure in JSON:

.. code-block:: json

   {
     "department": "MBS",
     "number": 337,
     "name": "Research Computing in Biology",
     "inperson": true,
     "finalgroups": null,
     "instructors": ["Kelsey", "Erik", "Joe"],
     "prerequisites": [
       {"course": "INB 321G", "name": "Principles of Computational Biology"},
       {"course": "BCH 339N", "name": "Systems Biology and Bioinformatics"},
       {"course": "CS 303E", "name": "Elements of Computers and Programming"}
     ]
   }

The same structure in YAML is:

.. code-block:: yaml

   ---
   department: MBS
   number: 337
   name: Research Computing in Biology
   inperson: true
   finalgroups: null         # can also use ~
   instructors:
     - Kelsey
     - Erik
     - Joe
   prerequisites:
     - course: INB 321G
       name: Principles of Computational Biology
     - course: SDS 322
       name: Systems Biology and Bioinformatics
     - course: CS 303E
       name: Elements of Computers and Programming
   ...

The whole thing can be considered a dictionary. The key ``instructors`` contains
a value that is a list of names, and the key ``prerequisites`` contains a value
that is a list of dictionaries. Booleans appear as ``false`` and ``true``
(lowercase only). Null / empty values appear as ``null`` or ``~``. And, as you
can see above, YAML also supports comments starting with a ``#``.

One glaring thing that is missing from the YAML file is quotation marks. In
general, you don't have to use quotes in YAML. You may use quotes to force a
number to be interpreted as a string (e.g. ``337`` will automatically be
interpreted as an integer, but ``"337"`` will be interpreted as a string).

.. note::

   Check out the list of uniprot proteins we worked with in the JSON section, but
   now in CSV format `here <https://raw.githubusercontent.com/kbeavers/mbs-337-sp26/refs/heads/main/docs/unit02/sample-data/uniprot_proteins_simple.yaml>`_.

There is a lot more to YAML, most of which we will not use in this course. Just
know that YAML files can contain:

* Comments
* Multi-line strings / text blocks
* Multi-word keys
* Complex objects
* Special characters
* Explicitly declared types
* A mechanism to duplicate / inherit values across a document ("anchors")


If we encounter a need for any of these, we can refer to the
`official YAML syntax <https://yaml.org/spec/1.2/spec.html>`_


Read YAML from File
-------------------

.. warning::

   There is no YAML interpreter in the Python 3 standard libary, so we need
   to install one with pip3. Make sure your virtual
   environment is activated:

   .. code-block:: console

      (myenv) [mbs-337]$ pip3 install pyyaml


Given the protein data in YAML format, which you can download from
`this link <https://raw.githubusercontent.com/kbeavers/mbs-337-sp26/refs/heads/main/docs/unit02/sample-data/uniprot_proteins_simple.yaml>`_,
load it into a Python3 dictionary object using the following:

.. code-block:: python3
   :linenos:

   import yaml

   data = {}

   with open("Protein_List.yaml", "r") as f:
      data = yaml.load(f, Loader = yaml.SafeLoader)
      print(data['protein_entries'][0])


Very similar to the JSON module, it only requires a few simple lines then you
have a dictionary object to work with. The ``Loader=yaml.SafeLoader`` parameter
makes it so no arbitrary Python code is executed when loading in the data - this
is typically a good choice for data from untrusted sources.


Write YAML to File
------------------

In a new script create a dictionary object that we can write to a new YAML file.

.. code-block:: python3
   :linenos:

   import yaml

   data = {}
   data['accession'] = 'PRJNA1412539'
   data['id'] = '1412539'
   data['title'] = 'Transposon-insertion sequencing uncovers nlpD as the essential gene for intracellular persistence and infectivity of Salmonella'
   data['dataType'] = 'Raw sequence reads'

   with open('data.yaml', 'w') as o:
       yaml.dump(data, o)

Notice that most of the code in the script above was simply assembling a normal
Python3 dictionary. The ``yaml.dump()`` method only requires two arguments - the
object that should be written to file, and the filehandle.

Inspect the output file. Do you notice anything interesting?

.. toggle:: Click to show

   1. You may notice that the keys are no longer in the same order as the ``data`` Python dictionary. This is because
   PyYAML sorts the keys lexicographically unless told otherwise. We can tell PyYAML to **not** sort keys like this:

   .. code-block:: python3

      with open('data.yaml', 'w') as o:
         yaml.dump(data, o, sort_keys=False)

   2. You may also notice that our output is missing the document start ``(---)`` and end ``(...)`` markers. Markers are typically
   only required to separate multiple documents in a single stream, making them technically unneccessary for our single-document
   output. We can explicitly add them like so:

   .. code-block:: python3

      with open('data.yaml', 'w') as o:
         yaml.dump(data, o, explicit_start=True, explicit_end=True)

Additional Resources
--------------------
* Many of the materials in this module were adapted from `COE 332: Software Engineering & Design <https://coe-332-sp26.readthedocs.io/en/latest/unit02/yaml.html>`_
* `YAML Spec <https://yaml.org/spec/1.2/spec.html>`_
* `YAML Validator <http://www.yamllint.com/>`_
* `JSON / YAML Converter <https://www.json2yaml.com/>`_
* `PyYAML Docs <https://pyyaml.org/wiki/PyYAMLDocumentation>`_
