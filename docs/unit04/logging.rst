Logging
=======

When you're writing Python scripts, you often need to see what the program is
doing—especially when something goes wrong. Up to this point, we've been using 
arbitraty ``print`` statements in our code as a way to debug errors. 
**Logging** is a better way to track what happens at runtime and allows us 
to diagnose what parts of our code are causing errors. 

After this module, you should be able to:

* Import the ``logging`` module and call ``logging.basicConfig()``
* Choose a log level: DEBUG, INFO, WARNING, ERROR, or CRITICAL
* Write messages at the right level
* Use logs to track down where warnings or errors came from

Why use logging instead of print?
---------------------------------

Logging has several advantages over using print statements for debugging and 
tracking any errors or misbehavior within our code. Logging allows you to specify the 
importance of messages (log levels), configure where the messages should be sent 
(console, log files, etc.), and more. 

* ``print()`` — Use for the normal output your program is *supposed* to
  show (e.g. results to the user).
* **Logging** — Use for everything else: extra detail for debugging,
  warnings that don't stop the program, errors that cause something to fail,
  and critical problems that prevent the program from continuing.

With logging, you can leave all those diagnostic messages in your code and
control how much you see by changing one setting (the log level). No need to
delete or comment out ``print`` statements.

Log levels
----------

**Log levels** indicate how important or severe a message is. Python's
``logging`` module (in the standard library) defines five standard levels,
from most to least verbose:

+-----------+---------------------------------------------------------------+
| Level     | Use it for                                                    |
+===========+===============================================================+
| DEBUG     | Detailed info for troubleshooting (e.g. "entering function X")|
+-----------+---------------------------------------------------------------+
| INFO      | Normal progress ("file opened", "processed 100 records")      |
+-----------+---------------------------------------------------------------+
| WARNING   | Something unexpected but the program keeps running            |
+-----------+---------------------------------------------------------------+
| ERROR     | A real problem; some operation failed                         |
+-----------+---------------------------------------------------------------+
| CRITICAL  | Very serious; the program may not be able to continue         |
+-----------+---------------------------------------------------------------+

If you set the log level to ``WARNING``, you'll see WARNING, ERROR, and
CRITICAL messages, but not DEBUG or INFO. If you set it to ``DEBUG``, you'll
see everything. That way you can leave DEBUG and INFO messages in your code
and only enable them when you need to dig into a problem.

(Source: `Python Logging How-To <https://docs.python.org/3/howto/logging.html>`_.)

Your first logging script
--------------------------

**Step 1: Minimal setup**

Create a script (e.g. ``log_test.py``) with:

.. code-block:: python3
   :linenos:

   import logging
   logging.basicConfig()    # turn on the default logging setup

   logging.debug('This is a DEBUG message')
   logging.info('This is an INFO message')
   logging.warning('This is a WARNING message')
   logging.error('This is an ERROR message')
   logging.critical('This is a CRITICAL message')

Run it:

.. code-block:: console

   $ python3 log_test.py
   WARNING:root:This is a WARNING message
   ERROR:root:This is an ERROR message
   CRITICAL:root:This is a CRITICAL message

By default, the log level is set to **WARNING**. So you only see WARNING and above;
DEBUG and INFO are hidden.

**Step 2: Show all messages**

To see DEBUG and INFO too, set the level when you call ``basicConfig``:

.. code-block:: python3
   :linenos:
   :emphasize-lines: 2

   import logging
   logging.basicConfig(level=logging.DEBUG)

   logging.debug('This is a DEBUG message')
   logging.info('This is an INFO message')
   logging.warning('This is a WARNING message')
   logging.error('This is an ERROR message')
   logging.critical('This is a CRITICAL message')

Now all five messages appear.

**Step 3: Set the level from the command line**

You can make the log level a command-line option so you don't have to edit
the script each time you want more or less output. To do this, we use **argparse**, 
which is Python's standard library module for reading options provided when a program 
is run from the command line (for example ``python3 log_test.py -l DEBUG``).

.. code-block:: python3
    :linenos:
    :emphasize-lines: 1,5,8-12,15,18

    import argparse
    import logging

    # Create a parser object responsible for reading/understanding command-line arguments
    parser = argparse.ArgumentParser()

    # Add one command-line option to the parser 
    parser.add_argument('-l', '--loglevel', 
                        type=str, 
                        required=False, 
                        default='WARNING',
                        help='set log level to DEBUG, INFO, WARNING, ERROR, or CRITICAL')

    # Create object that will store command-line arguments provided by the user 
    args = parser.parse_args()

    # Use user-defined log level 
    logging.basicConfig(level=args.loglevel)

    logging.debug('This is a DEBUG message')
    logging.info('This is an INFO message')
    logging.warning('This is a WARNING message')
    logging.error('This is an ERROR message')
    logging.critical('This is a CRITICAL message')

In this example, we first create a *parser* object using ``argparse.ArgumentParser()``. 
Then we use ``parser.add_argument(...)`` to tell the parser about **one command-line option** 
that our program supports. In this case, we define a log level option:

* ``-l`` and ``--loglevel``: Two ways o specify the same option (short and long forms)
* ``type=str``: The value we pass via the command-line should be read as a string (e.g., 'DEBUG')
* ``required=False``: The user does not have to provide this option
* ``default='WARNING'``: If the user does not provide a log level, use WARNING automatically 
* ``help='...'``: Text that is shown when the user runs ``--help`` 

Then, we create an object called ``args`` that will store the command-line options provided by the user. 
These are stored using dot notation (e.g., ``args.loglevel``), so we then pass this value to 
``basicConfig``.

Examples:

.. code-block:: console

   $ python3 log_test.py
   # only WARNING and above (default)

   $ python3 log_test.py -l DEBUG
   # all messages

   $ python3 log_test.py --loglevel ERROR
   # only ERROR and CRITICAL

Making log messages more useful
-------------------------------

Plain messages are okay, but in real projects (and especially when you run
the same program on several machines) it helps to include:

* **When** the event happened (timestamp)
* **Where** it happened (script name, function, line number)
* **Which machine** (hostname), if you have multiple servers or containers

You can do that by passing a **format string** to ``basicConfig`` and, if you
want the hostname, using the ``socket`` module. The ``socket`` module lets us 
ask the operating system for information regarding the system we're running on. 
For example, we can use ``socket.gethostname()``, which returns the name of the 
machine the program is running on. 

Here's an example format string: 

.. code-block:: python3 

    format_str = (
        f'[%(asctime)s {socket.gethostname()}] '
        '%(filename)s:%(funcName)s:%(lineno)s - %(levelname)s: %(message)s'
    )








Here's an example:

.. code-block:: python3
    :linenos:
    :emphasize-lines: 3,10,11

    import argparse
    import logging
    import socket

    parser = argparse.ArgumentParser()
    parser.add_argument('-l', '--loglevel',
                        type=str,
                        required=False,
                        default='WARNING',
                        help='set log level to DEBUG, INFO, WARNING, ERROR, or CRITICAL')
    args = parser.parse_args()

    format_str = (
        f'[%(asctime)s {socket.gethostname()}] '
        '%(filename)s:%(funcName)s:%(lineno)s - %(levelname)s: %(message)s'
    )
    logging.basicConfig(level=args.loglevel, format=format_str)

    logging.debug('This is a DEBUG message')
    logging.info('This is an INFO message')
    logging.warning('This is a WARNING message')
    logging.error('This is an ERROR message')
    logging.critical('This is a CRITICAL message')

The format uses placeholders the logging module understands: ``%(asctime)s``
(time), ``%(filename)s``, ``%(funcName)s``, ``%(lineno)s``, ``%(levelname)s``,
``%(message)s``. A table explaining these attributes and more can be found 
`here <https://docs.python.org/3/library/logging.html#logrecord-attributes>`_.

Your output might look like:

.. code-block:: console

    $ python3 log_test.py
    [2026-02-09 21:16:06,512 mbs-337-14] log_test.py:<module>:18 - WARNING: This is a WARNING message
    [2026-02-09 21:16:06,512 mbs-337-14] log_test.py:<module>:19 - ERROR: This is an ERROR message
    [2026-02-09 21:16:06,512 mbs-337-14] log_test.py:<module>:20 - CRITICAL: This is a CRITICAL message


Exercise: Add logging to the FASTQ summary script
------------------------------------------------

The script below reads a FASTQ file, summarizes each read, and writes the
results to JSON. **Add logging** to it, with emphasis on:

* **DEBUG** — e.g. when you open a file, start processing, or finish a step
* **ERROR** — when something goes wrong (e.g. file not found, invalid data)

Use the ``logging`` module and ``basicConfig``. Optionally support a
command-line log level (e.g. ``-l DEBUG``) and a format that includes
timestamp and location.

Starter script (no logging yet):

.. code-block:: python3
   :linenos:

   import json
   from Bio import SeqIO
   from models import ReadSummary, FastqSummary

   # -------------------------
   # Constants (configuration)
   # -------------------------
   FASTQ_FILE = 'raw_reads.fastq'
   OUTPUT_JSON = 'fastq_summary.json'
   ENCODING = 'fastq-sanger'

   # -------------------------
   # Functions
   # -------------------------
   def summarize_record(record) -> ReadSummary:
       # Convert one FASTQ record into a ReadSummary instance

       phred_scores = record.letter_annotations['phred_quality']
       average_phred = sum(phred_scores) / len(phred_scores)

       return ReadSummary(
           id=record.id,
           sequence=str(record.seq),
           total_bases=len(record.seq),
           average_phred=round(average_phred, 2)
       )

   def summarize_fastq_file(fastq_file: str, encoding: str) -> FastqSummary:
       # Read a FASTQ file and return a FastqSummary instance

       reads_list = []

       with open(fastq_file, 'r') as f:
           for record in SeqIO.parse(f, encoding):
               reads_list.append(summarize_record(record))

       return FastqSummary(reads=reads_list)

   def write_summary_to_json(summary: FastqSummary, output_file: str) -> None:
       # Write FastqSummary to a JSON file

       with open(output_file, 'w') as outfile:
           json.dump(summary.model_dump(), outfile, indent=2)

   def main():
       summary = summarize_fastq_file(FASTQ_FILE, ENCODING)
       write_summary_to_json(summary, OUTPUT_JSON)

   if __name__ == '__main__':
       main()

.. toggle:: Click to see the solution

   One way to add DEBUG and ERROR logging, plus optional command-line log
   level and a clear format:

   .. code-block:: python3
      :linenos:

      import argparse
      import json
      import logging
      from pathlib import Path

      from Bio import SeqIO
      from models import ReadSummary, FastqSummary

      # -------------------------
      # Constants (configuration)
      # -------------------------
      FASTQ_FILE = 'raw_reads.fastq'
      OUTPUT_JSON = 'fastq_summary.json'
      ENCODING = 'fastq-sanger'

      # -------------------------
      # Functions
      # -------------------------
      def summarize_record(record) -> ReadSummary:
          phred_scores = record.letter_annotations['phred_quality']
          average_phred = sum(phred_scores) / len(phred_scores)
          return ReadSummary(
              id=record.id,
              sequence=str(record.seq),
              total_bases=len(record.seq),
              average_phred=round(average_phred, 2)
          )

      def summarize_fastq_file(fastq_file: str, encoding: str) -> FastqSummary:
          reads_list = []
          if not Path(fastq_file).exists():
              logging.error('Input file not found: %s', fastq_file)
              raise FileNotFoundError(f'Input file not found: {fastq_file}')

          logging.debug('Opening FASTQ file: %s', fastq_file)
          with open(fastq_file, 'r') as f:
              for i, record in enumerate(SeqIO.parse(f, encoding)):
                  reads_list.append(summarize_record(record))
                  if (i + 1) % 100 == 0:
                      logging.debug('Processed %d reads', i + 1)

          logging.debug('Finished reading %d reads from %s', len(reads_list), fastq_file)
          return FastqSummary(reads=reads_list)

      def write_summary_to_json(summary: FastqSummary, output_file: str) -> None:
          try:
              logging.debug('Writing summary to %s (%d reads)', output_file, len(summary.reads))
              with open(output_file, 'w') as outfile:
                  json.dump(summary.model_dump(), outfile, indent=2)
              logging.debug('Successfully wrote %s', output_file)
          except OSError as e:
              logging.error('Failed to write %s: %s', output_file, e)
              raise

      def main():
          parser = argparse.ArgumentParser()
          parser.add_argument('-l', '--loglevel', type=str, default='WARNING',
                              help='Log level: DEBUG, INFO, WARNING, ERROR, CRITICAL')
          args = parser.parse_args()
          logging.basicConfig(
              level=getattr(logging, args.loglevel.upper(), logging.WARNING),
              format='%(asctime)s - %(levelname)s - %(filename)s:%(lineno)s - %(message)s'
          )

          summary = summarize_fastq_file(FASTQ_FILE, ENCODING)
          write_summary_to_json(summary, OUTPUT_JSON)

      if __name__ == '__main__':
          main()

   **What this solution does:**

   * **DEBUG** — Logs when the FASTQ file is opened, progress every 100 reads,
     total reads read, and when writing the JSON file (start and success).
   * **ERROR** — Logs when the input file is missing and when writing the
     output file fails; then re-raises so the program still fails visibly.
   * **Command-line** — ``-l DEBUG`` (or ``--loglevel DEBUG``) shows all
     debug messages; default remains WARNING.
   * **Format** — Uses timestamp, level, filename:line, and message so you can
     trace where each log line came from.

   You can add more DEBUG lines (e.g. per-read in a small file) or INFO
   messages for normal milestones; the pattern is the same.

Additional resources
--------------------

* Materials adapted from `COE 332: Software Engineering & Design
  <https://coe-332-sp26.readthedocs.io/en/latest/unit03/documentation.html>`_
* `Python Logging How-To <https://docs.python.org/3/howto/logging.html>`_
