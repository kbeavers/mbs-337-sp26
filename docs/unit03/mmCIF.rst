Working with Structure Data (PDBx/mmCIF)
====================================

In this hands-on module, we will learn how to work with the mmCIF data format.
mmCIF is the modern, machine-friendly format used by the `RCSB Protein Data Bank (PDB) <https://www.rcsb.org/>`_
to store 3D macromolecular structures and their associated metadata.

mmCIF files are used to represent experimentally determined and computationally
predicted structures, including proteins, nucleic acids, and molecular complexes.

After completing this module, students should be able to:

* Identify and understand valid mmCIF files
* Explain what types of information mmCIF stores (atoms, residues, chains, metadata)
* Read mmCIF files into Python objects using Biopython
* Navigate the structure hierarchy (Structure → Model → Chain → Residue → Atom)
* Extract common data (coordinates, residue IDs, chain IDs, B-factors, occupancy)
* Write or export structures back to mmCIF format

Macromolecular structure context 
--------------------------

mmCIF files describe **macromolecular structures**, meaning the three-dimensional
positions of atoms in a biological system. While many entries in the Protein Data Bank
are proteins, mmCIF is not protein-specific. It can represent a wide range of
structures, including:

* Proteins and protein complexes
* DNA and RNA
* Protein–nucleic acid complexes
* Small-molecule ligands, ions, cofactors, and waters

Biological function depends strongly on 3D shape. Binding sites, active sites,
molecular recognition, and molecular interactions are determined by how atoms and
residues are arranged in space. Structural data come from experiments such as X-ray
crystallography, NMR spectroscopy, and cryo-electron microscopy, or from computational
prediction methods such as `AlphaFold <https://alphafold.ebi.ac.uk/>`_.

The Protein Data Bank archives these structures so they can be analyzed, visualized,
and compared.

.. figure:: images/RCSB_1MBN.png
    :width: 1000px
    :align: center

    Structure summary of PDB Entry `1MBN <https://www.rcsb.org/structure/1MBN>`_.

    *Fun fact: this was the first protein to have its 3D structure revealed by
    X-ray crystallography. John Kendrew was awarded the Nobel Prize in Chemistry 
    in 1962 for this achievement.*

What is mmCIF?
-------------------

**PDBx/mmCIF**, or simply **mmCIF**, stands for "Macromolecular CIF". This name is derived from 
the "Crystallographic Information File (CIF)" format, which was created to store small molecule 
crystallographic experiments. mmCIF uses the same syntax and file extension as CIF (``.cif``), 
but uses a different, much larger, and extensible dictionary. 

.. note::

    Note: The PDB now exclusively uses mmCIF (PDBx/mmCIF) for archiving data, 
    making it the essential format for modern structural biology, whereas standard 
    CIF is mostly used in chemical crystallography. 


A typical mmCIF contains:

* Atom-level coordinates (x, y, z) for each atom
* Atom identity (element, atom name), residue identity (amino acid), chain identity
* Optional values like occupancy and B-factor (temperature factor)
* Experimental metadata (method, resolution, authors, citations)
* Chemical component definitions (ligands, modified residues)

mmCIF Format Basics
-------------------

While mmCIF files can be intimidating at first, they are built from just a few core concepts.
At a high level, an mmCIF file consits of:

* **Data Blocks**: contain information for one structure (some mmCIF files contain multiple structures)
* **Data items**: single name-value pairs for metadata fields 
* **Loops**: contains tables with rows corresponding to column names (more on this below)

Data Blocks
+++++++++++++++

Every mmCIF file begins with a **data block**, which is introduced by a line starting with ``data_``:

.. code-block:: text

    data_1MBN

The text after ``data_`` is the data block identifier, usually the PDB ID (e.g. "1MBN").
A data block defines a logical boundary for one structure entry. 

Important rules:

* A file may contain one or more data blocks
* Each data block describes one structure
* A data block ends when another ``data_`` line is encountered or the file ends

Data Items (Name-Value Pairs)
++++++++++++++++++++++++++++++++

The simplest information in a mmCIF file is stored as **data items**, which are single
name-value pairs:

.. code-block:: text

    _struct.title   'The stereochemistry of the protein myoglobin'

This line has three parts:

* ``_struct.title``: The data item **name**
* ``'The stereochemistry of the protein myoglobin'``: The **value**
* Whitespace separates name from values 

All mmCIF data item names begin with a leading underscore and have the form
``_category.keyword``. In the example above:

* ``_struct`` = the category 
* ``title`` = the keyword (which must be unique within the category)

.. note:: Data Items Syntax:

    * Data item names are not case-sensitive

    * Each data item must have **exactly one value**

    * Values may be numbers, short strings (quoted or unquoted), or special placeholders:

       * ``?`` = value is missing
       * ``.`` = value is not applicable or intentially omitted 

**Text Values and Multi-line Strings**

Short text values may be enclosed in single or double quotes, or can be unquoted:

.. code-block:: text

    _entity_src_gen.gene_src_common_name               'sperm whale' 
    _entity_src_gen.gene_src_genus                     Physeter 
    _entity_src_gen.pdbx_gene_src_scientific_name      'Physeter catodon' 

Long text values that span multiple lines are enclosed by semicolon delimiters 
placed at the start of a line:

.. code-block:: text

    _entity_poly.pdbx_seq_one_letter_code       
    ;VLSEGEWQLVLHVWAKVEADVAGHGQDILIRLFKSHPETLEKFDRFKHLKTEAEMKASEDLKKHGVTVLTALGAILKKKG
    HHEAELKPLAQSHATKHKIPIKYLEFISEAIIHVLHSRHPGDFGADAQGAMNKALELFRKDIAAKYKELGYQG
    ;

Everything between the semicolons is treated as a single text value. 

Loops (Tables)
+++++++++++++++++

Many types of data occur multiple times (atoms, residues, authors, citations, etc.). 
These are stored using the ``loop_`` directive, which defines a table. 
A loop has two parts:

1. A list of column names (data item names)
2. Rows of values corresponding to those columns

For example, let's take a look at the simplified ``_atom_site`` loop. This is often the 
most crucial information contained in a mmCIF file:

.. code-block:: text

    loop_
    _atom_site.label_atom_id 
    _atom_site.label_comp_id 
    _atom_site.Cartn_x 
    _atom_site.Cartn_y 
    _atom_site.Cartn_z 
    N  VAL  -2.900  17.600  15.500  
    C  VAL  -3.600  16.400  15.300
    C  VAL  -3.000  15.300  16.200
    O  VAL  -3.700  14.700  17.000

The first line (``loop_``) starts the table. Then, the lines starting with ``_atom_site.`` define the 
**columns** of the table:
 * ``label_atom_id``: The atom name within the residue (e.g., N, C, O)
 * ``label_comp_id``: The residue name (here, VAL = valine)
 * ``Cartn_x``, ``Cartn_y``, ``Cartn_z``: The atom's x, y, and z coordinates in 3D space (in Ångströms)

Each line below the headers provides values for **one atom**, in the same order as the column header. 
For example, the first row:

.. code-block:: text

    N  VAL  -2.900  17.600  15.500  

means: Atom N (the backbone nitrogen) in residue VAL (valine) is located at coordinates (−2.900, 17.600, 15.500) Å


Working with mmCIF files
------------------------

Let's get some practice working with mmCIF files. We'll use VSCode for these exercises.
Open a VSCode RemoteSSH session and create a new terminal.

Within the terminal inside VSCode on your class VM, navigate to your ``mbs-337`` project directory
and create a new directory called ``working-with-mmCIF``.

We're going to use a new command called ``wget`` to download a mmCIF file directly from 
`RCSB PDB <https://www.rcsb.org/>`_. This command allows us to retrieve files directly from the internet
by providing a URL to the data we want to download. 

Within your VS Code terminal, use the below command to download the mmCIF file for PDB Entry `1MBN <https://www.rcsb.org/structure/1MBN>`_.

.. code-block:: console

    [mbs-337]$ wget https://files.rcsb.org/download/1MBN.cif.gz

.. code-block:: console 

    --2026-02-02 21:44:40--  https://files.rcsb.org/download/1MBN.cif.gz
    Resolving files.rcsb.org (files.rcsb.org)... 13.33.82.83, 13.33.82.18, 13.33.82.74, ...
    Connecting to files.rcsb.org (files.rcsb.org)|13.33.82.83|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [application/gzip]
    Saving to: ‘1MBN.cif.gz’

    1MBN.cif.gz                               [ <=>     ]  36.54K  --.-KB/s    in 0.02s   

    2026-02-02 21:44:40 (2.15 MB/s) - ‘1MBN.cif.gz’ saved [37416]

You now have a file called ``1MBN.cif.gz`` in your ``working-with-bio-data`` directory!
If you try to open this file, you'll see that it does not display in the text editor. That's 
because we downloaded a compressed version of the mmCIF — the ``.gz`` file ending means that
this file has been compressed using the GNU Zip (Gzip) algorithm to reduce the file size
for storage and faster transfer. 

The first thing we need to do is decompress this file to its original format:

.. code-block:: console

    [mbs-337]$ gunzip 1MBN.cif.gz

The ``.gz`` file ending should be gone now, and you should be able to view your mmCIF
file in the text editor.

Helpful Linux Commands
~~~~~~~~~~~~~~~~~~~~~~

You can inspect an mmCIF file from the command line:

.. code-block:: bash

    # Print the first 20 lines
    [mbs-337]$ head -n 20 1MBN.cif

    # Find the title of the structure
    [mbs-337]$ grep "_struct.title" 1MBN.cif

    # Find lines that start a data block
    [mbs-337]$ grep "^data_" 1MBN.cif

Explore the mmCIF file with Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before using a parser, it helps to see what the file actually contains. As in the CSV
and FASTA modules, we open the file and work with its contents. Here we read the ``.cif``
as plain text to see the **data block**, **key–value** lines, and **loop_** tables.

Create a short script (e.g. ``explore_cif.py``) that (1) prints the first 20 lines—like
``head -n 20``—and (2) collects category names. In mmCIF, any line that starts with
``_`` and contains ``.`` has a category name before the dot (e.g. ``_atom_site.label_atom_id``
→ category ``_atom_site``).

.. code-block:: python3

    with open('1MBN.cif', 'r') as f:
        lines = f.readlines()

    # Step 1: Print the first 20 lines
    for line in lines[:20]:
        print(line, end='')

    # Step 2: Collect category names (one per unique _category)
    categories = set()
    for line in lines:
        line = line.strip()
        if line.startswith('_') and '.' in line:
            category = line.split('.')[0]
            categories.add(category)

    print("Number of categories:", len(categories))
    print("First 10 categories:", sorted(categories)[:10])

**What this does:** We use ``readlines()`` to get a list of lines (as in working with
other text files). We print the first 20 with a slice ``lines[:20]``. Then we loop over
all lines: if a line starts with ``_`` and contains ``.``, the part before the dot is
a category name; we add it to a set so each category is counted once. At the end we
print the count and the first 10 category names.

.. code-block:: console

    data_1MBN
    #
    _entry.id   1MBN
    _entry.name   ?
    ...
    Number of categories: 42
    First 10 categories: ['_atom_site', '_atom_site_anisotrop', '_audit_author', ...]

So the file really does start with a **data block** (``data_1MBN``), then **key–value**
entries (e.g. ``_entry.id   1MBN``), and later **loop_** sections with table headers
and rows. The ``_atom_site`` category is the one that holds the atomic coordinates;
Biopython’s parser will turn that (and related categories) into the Structure →
Model → Chain → Residue → Atom hierarchy we use next.

Read mmCIF from File
~~~~~~~~~~~~~~~~~~~~~

Biopython provides **MMCIFParser** in ``Bio.PDB`` for reading ``.cif`` files. The parser
converts the text file into a **Structure** object that you can query and iterate over.
Activate your Python virtual environment and create a file called ``mmcif_ex.py``:

.. code-block:: python3

    from Bio.PDB.MMCIFParser import MMCIFParser

    parser = MMCIFParser()

    with open('1MBN.cif', 'r') as f:
        structure = parser.get_structure('myoglobin', f)

The ``get_structure()`` method takes two inputs:

1. **An ID** — a short name you choose (e.g. ``'myoglobin'``)
2. **A file handle** — the open mmCIF file

Understanding the structure hierarchy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Biopython represents a structure using a hierarchy:

* **Structure** (top-level, one per file)
  * **Model** (some files contain multiple models, e.g. NMR ensembles)
    * **Chain** (e.g. chain A, B)
      * **Residue** (amino acids: GLY, ALA, VAL, ...)
        * **Atom** (individual atoms with 3D coordinates)

You typically iterate with nested loops over ``structure`` → ``model`` → ``chain`` →
``residue`` → ``atom``. For example, to count chains, residues, and atoms:

.. code-block:: python3

    num_chains = 0
    num_residues = 0
    num_atoms = 0

    for model in structure:
        for chain in model:
            num_chains += 1
            for residue in chain:
                num_residues += 1
                for atom in residue:
                    num_atoms += 1

    print(f"Chains: {num_chains}")
    print(f"Residues: {num_residues}")
    print(f"Atoms: {num_atoms}")

.. code-block:: console

    Chains: 1
    Residues: 153
    Atoms: 1210

Each object has an ``id`` (e.g. ``chain.get_id()`` returns ``'A'``, ``residue.get_resname()``
returns the three-letter code). Atoms have ``.get_coord()`` for x, y, z coordinates. You
can use this same pattern to filter chains, extract sequences, or work with coordinates.

**Connecting the file structure to the hierarchy**

When we explored the raw mmCIF file, we saw a **data block** (``data_1MBN``), **key–value**
entries (e.g. ``_struct.title``), and **loop_** tables such as ``_atom_site`` with columns
for atom IDs, residue names, chain IDs, and coordinates. Biopython’s ``MMCIFParser`` reads
those categories and builds the **Structure → Model → Chain → Residue → Atom** hierarchy.
So the same information you saw in the text file (atoms, residues, chains) is now
accessible as Python objects you can iterate over and query. Understanding both the
raw mmCIF structure and the Biopython hierarchy helps when you need to extract specific
data (e.g. B-factors, occupancy) or write structures back to mmCIF.

.. tip::

    mmCIF files can be large. The parser loads the structure into memory so you can query
    it easily. For most single-protein structures this is fine; very large complexes may
    require more memory or streaming approaches.

EXERCISE
---------

Using the same structure file (e.g. ``1MBN.cif``), write a short script that:

1. Parses the mmCIF file with ``MMCIFParser``.
2. Loops over models and chains.
3. For each chain, prints the **chain ID** and the **number of residues** in that chain.

Example output: ``Chain A: 153 residues``.

.. toggle:: Click to see the solution

    .. code-block:: python3
        :linenos:

        from Bio.PDB.MMCIFParser import MMCIFParser

        parser = MMCIFParser()
        with open('1MBN.cif', 'r') as f:
            structure = parser.get_structure('myoglobin', f)

        for model in structure:
            for chain in model:
                chain_id = chain.get_id()
                num_residues = len(list(chain.get_residues()))
                print(f"Chain {chain_id}: {num_residues} residues")

In this exercise, we used the same hierarchy (structure → model → chain → residue). We
get the chain identifier with ``chain.get_id()`` and count residues by iterating over
``chain.get_residues()`` (or ``list(chain)``). From here you can extend to extracting
residue names for a sequence, or atom coordinates for distance calculations.


Additional Resources
--------------------

* `PDB-101: Beginner's Guide to PDBx/mmCIF <https://pdb101.rcsb.org/learn/guide-to-understanding-pdb-data/beginner%E2%80%99s-guide-to-pdbx-mmcif>`_ — format syntax and examples
* `RCSB PDB <https://www.rcsb.org/>`_ — search and download structures (mmCIF)
* `AlphaFold Protein Structure Database <https://alphafold.ebi.ac.uk/>`_ — predicted structures (mmCIF)
* `Biopython PDB module <https://biopython.org/wiki/The_Biopython_Structural_Biology_FAQ>`_ — parsing structures
