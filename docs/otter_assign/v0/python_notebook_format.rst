Python Notebook Format
======================

Otter's notebook format groups prompts, solutions, and tests together into questions. Autograder tests 
are specified as cells in the notebook and their output is used as the expected output of the 
autograder when genreating tests. Each question has metadata, expressed in a code block in YAML 
format when the question is declared. Tests generated by Otter Assign follow the :ref:`Otter-
compliant OK format <test_files>`.

**Note:** Otter Assign is also backwards-compatible with jAssign-formatted notebooks. For more 
information about formatting notebooks for jAssign, see `its documentation
<https://github.com/okpy/jassign/blob/master/docs/notebook-format.md>`_.


Assignment Metadata
-------------------

In addition to various command line arugments discussed below, Otter Assign also allows you to 
specify various assignment generation arguments in an assignment metadata cell. These are very 
similar to the question metadata cells described in the next section. Assignment metadata, included 
by convention as the first cell of the notebook, places YAML-formatted configurations inside a code 
block that begins with ``BEGIN ASSIGNMENT``:

.. code-block:: markdown

    ```
    BEGIN ASSIGNMENT
    init_cell: false
    export_cell: true
    ...
    ```

This cell is removed from both output notebooks. These configurations, listed in the YAML snippet 
below, can be **overwritten** by their command line counterparts (e.g. ``init_cell: true`` is 
overwritten by the ``--no-init-cell`` flag). The options, their defaults, and descriptions are 
listed below. Any unspecified keys will keep their default values. For more information about many 
of these arguments, see :ref:`otter_assign_usage`. Any keys that map to 
sub-dictionaries (e.g. ``export_cell``, ``generate``) can have their behaviors turned off by 
changing their value to ``false``. The only one that defaults to true (with the specified sub-key 
defaults) is ``export_cell``.

.. BEGIN YAML TARGET: otter.assign.v0.assignment._DEFAULT_ASSIGNMENT_CONFIGURATIONS_WITH_DESCRIPTIONS

.. code-block:: yaml

    requirements: null             # the path to a requirements.txt file
    overwrite_requirements: false  # whether to overwrite Otter's default requirement.txt in Otter Generate
    environment: null              # the path to a conda environment.yml file
    run_tests: true                # whether to run the assignment tests against the autograder notebook
    solutions_pdf: false           # whether to generate a PDF of the solutions notebook
    template_pdf: false            # whether to generate a filtered Gradescope assignment template PDF
    init_cell: true                # whether to include an Otter initialization cell in the output notebooks
    check_all_cell: true           # whether to include an Otter check-all cell in the output notebooks
    export_cell:                   # whether to include an Otter export cell in the output notebooks
      instructions: ''             # additional submission instructions to include in the export cell
      pdf: true                    # whether to include a PDF of the notebook in the generated zip file
      filtering: true              # whether the generated PDF should be filtered
      force_save: false            # whether to force-save the notebook with JavaScript (only works in classic notebook)
      run_tests: false             # whether to run student submissions against local tests during export
    seed: null                     # a seed for intercell seeding
    generate: false                # grading configurations to be passed to Otter Generate as an otter_config.json; if false, Otter Generate is disabled
    save_environment: false        # whether to save the student's environment in the log
    variables: {}                  # a mapping of variable names to type strings for serlizing environments
    ignore_modules: []             # a list of modules to ignore variables from during environment serialization
    files: []                      # a list of other files to include in the output directories and autograder
    autograder_files: []           # a list of other files only to include in the autograder
    plugins: []                    # a list of plugin names and configurations
    test_files: true               # whether to store tests in separate .py files rather than in the notebook metadata
    colab: false                   # whether this assignment will be run on Google Colab

.. END YAML TARGET

All paths specified in the configuration should be **relative to the directory containing the master 
notebook**. If, for example, I was running Otter Assign on the ``lab00.ipynb`` notebook in the 
structure below:

.. code-block::

    dev
    ├── lab
    │   └── lab00
    │       ├── data
    │       │   └── data.csv
    │       ├── lab00.ipynb
    │       └── utils.py
    └── requirements.txt

and I wanted my requirements from ``dev/requirements.txt`` to be include, my configuration would 
look something like this:

.. code-block:: yaml

    requirements: ../../requirements.txt
    files:
        - data/data.csv
        - utils.py
    ...

A note about Otter Generate: the ``generate`` key of the assignment metadata has two forms. If you 
just want to generate and require no additional arguments, set ``generate: true`` in the YAML and 
Otter Assign will simply run ``otter generate`` from the autograder directory (this will also 
include any files passed to ``files``, whose paths should be **relative to the directory containing 
the notebook**, not to the directory of execution). If you require additional arguments, e.g. 
``points`` or ``show_stdout``, then set ``generate`` to a nested dictionary of these parameters and 
their values:

.. code-block:: yaml

    generate:
        seed: 42
        show_stdout: true
        show_hidden: true

You can also set the autograder up to automatically upload PDFs to student submissions to another 
Gradescope assignment by setting the necessary keys in ``generate``:

.. code-block:: yaml

    generate:
        token: ''
        course_id: 1234        # required
        assignment_id: 5678    # required
        filtering: true        # true is the default

If you don't specify a token, you will be prompted for your username and password when you run Otter
Assign; optionally, you can specify these via the command line with the ``--username`` and 
``--password`` flags. You can also run the following to retrieve your token:

.. code-block:: python

    from otter.generate.token import APIClient
    print(APIClient.get_token())

Any configurations in your ``generate`` key will be put into an ``otter_config.json`` and used when
running Otter Generate.

If you are grading from the log or would like to store students' environments in the log, use the 
``save_environment`` key. If this key is set to ``true``, Otter will serialize the stuednt's 
environment whenever a check is run, as described in :ref:`logging`. To restrict the 
serialization of variables to specific names and types, use the ``variables`` key, which maps 
variable names to fully-qualified type strings. The ``ignore_modules`` key is used to ignore 
functions from specific modules. To turn on grading from the log on Gradescope, set 
``generate[grade_from_log]`` to ``true``. The configuration below turns on the serialization of 
environments, storing only variables of the name ``df`` that are pandas dataframes.

.. code-block:: yaml

    save_environment: true
    variables:
        df: pandas.core.frame.DataFrame

As an example, the following assignment metadata includes an export cell but no filtering, no init 
cell, and passes the configurations ``points`` and ``seed`` to Otter Generate via the 
``otter_config.json``.

.. code-block:: markdown

    ```
    BEGIN ASSIGNMENT
    export_cell:
        filtering: false
    init_cell: false
    generate:
        points: 3
        seed: 0
    ```


Autograded Questions
--------------------

Here is an example question in an Otter Assign-formatted notebook:

.. raw:: html

    <iframe src="../../_static/notebooks/html/assign-code-question.html"></iframe>


For code questions, a question is a description *Markdown* cell, followed by a solution *code* cell 
and zero or more test *code* cells. The description cell must contain a code block (enclosed in 
triple backticks ```````) that begins with ``BEGIN QUESTION`` on its own line, followed by YAML 
that defines metadata associated with the question.

The rest of the code block within the description cell must be YAML-formatted with the following 
fields (in any order):

.. code-block:: yaml

    name: null        # (required) the path to a requirements.txt file
    manual: false     # whether this is a manually-graded question
    points: null      # how many points this question is worth; defaults to 1 internally
    check_cell: true  # whether to include a check cell after this question (for autograded questions only)

As an example, the question metadata below indicates an autograded question ``q1`` worth 1 point.

.. code-block:: markdown

    ```
    BEGIN QUESTION
    name: q1
    manual: false
    ```


Question Points
+++++++++++++++

The ``points`` key of the question metadata defines how many points each autograded question is 
worth. Note that the value specified here will be divided evenly among each test case you define for 
the question. Test cases are defined by the test cells you create (one test cell is one test case). 
So if you have three test cells and the question is worth 1 point (the default), each test case is 
worth 1/3 point and students will earn partial credit on the question by according to the proportion 
of test cases they pass.

Note that you can also define a point value for each individual test case by setting ``points`` to 
a dictionary with a single key, ``each``:

.. code-block:: yaml

    points:
        each: 1

or by setting ``points`` to a list of point values. The length of this list must equal the number of 
test cases, public and hidden, that correspond to this test case.

.. code-block:: yaml

    points:
        - 0
        - 1
        - 0.5
        # etc.


.. _otter_assign_python_solution_removal:

Solution Removal
++++++++++++++++

Solution cells contain code formatted in such a way that the assign parser replaces lines or portions 
of lines with prespecified prompts. Otter uses the same solution replacement rules as jAssign. From 
the `jAssign docs <https://github.com/okpy/jassign/blob/master/docs/notebook-format.md>`_:


* A line ending in ``# SOLUTION`` will be replaced by ``...``, properly indented. If
  that line is an assignment statement, then only the expression(s) after the
  ``=`` symbol will be replaced.
* A line ending in ``# SOLUTION NO PROMPT`` or ``# SEED`` will be removed.
* A line ``# BEGIN SOLUTION`` or ``# BEGIN SOLUTION NO PROMPT`` must be paired with
  a later line ``# END SOLUTION``. All lines in between are replaced with ``...`` or
  removed completely in the case of ``NO PROMPT``.
* A line ``""" # BEGIN PROMPT`` must be paired with a later line ``""" # END
  PROMPT``. The contents of this multiline string (excluding the ``# BEGIN
  PROMPT``) appears in the student cell. Single or double quotes are allowed.
  Optionally, a semicolon can be used to suppress output: ``"""; # END PROMPT``

.. code-block:: python

    def square(x):
        y = x * x # SOLUTION NO PROMPT
        return y # SOLUTION

    nine = square(3) # SOLUTION

would be presented to students as

.. code-block:: python

    def square(x):
        ...

    nine = ...

And

.. code-block:: python

    pi = 3.14
    if True:
        # BEGIN SOLUTION
        radius = 3
        area = radius * pi * pi
        # END SOLUTION
        print('A circle with radius', radius, 'has area', area)

    def circumference(r):
        # BEGIN SOLUTION NO PROMPT
        return 2 * pi * r
        # END SOLUTION
        """ # BEGIN PROMPT
        # Next, define a circumference function.
        pass
        """; # END PROMPT

would be presented to students as

.. code-block:: python

    pi = 3.14
    if True:
        ...
        print('A circle with radius', radius, 'has area', area)

    def circumference(r):
        # Next, define a circumference function.
        pass


Test Cells
++++++++++

There are two ways to format test cells. The test cells are any code cells following the solution 
cell that begin with the comment ``## Test ##`` or ``## Hidden Test ##`` (case insensitive). A 
``Test`` is distributed to students so that they can validate their work. A ``Hidden Test`` is not 
distributed to students, but is used for scoring their work.

Test cells also support test case-level metadata. If your test requires metadata beyond whether the 
test is hidden or not, specify the test by including a mutliline string at the top of the cell that 
includes YAML-formatted test metadata. For example,

.. code-block:: python

    """ # BEGIN TEST CONFIG
    points: 1
    success_message: Good job!
    """ # END TEST CONFIG
    do_something()

The test metadata supports the following keys with the defaults specified below:

.. code-block:: yaml

    hidden: false          # whether the test is hidden
    points: null           # the point value of the test
    success_message: null  # a messsge to show to the student when the test case passes
    failure_message: null  # a messsge to show to the student when the test case fails

Because points can be specified at the question level and at the test case level, point values get 
resolved as follows:

* If one or more test cases specify a point value and no point value is specified for the question, 
  each test case with unspecified point values is assumed to be worth 0 points.
* If one or more test cases specify a point value and a point value *is* specified for the question, 
  each test case with unspecified point values is assumed to be equally weighted and together are 
  worth the question point value less the sum of specified point values. For example, in a 6-point 
  question with 4 test cases where two test cases are each specified to be worth 2 points, each of 
  the other test cases is worth :math:`\frac{6-(2 + 2)}{2} = 1` point.)
* If no test cases specify a point value and a point value *is* specified for the question, each 
  test case is assumed to be equally weighted and is assigned a point value of :math:`\frac{p}{n}` 
  where :math:`p` is the number of points for the question and :math:`n` is the number of test 
  cases.
* If no test cases specify a point value and no point value is specified for the question, the 
  question is assumed to be worth 1 point and each test case is equally weighted.

**Note:** Currently, the conversion to OK format does not handle multi-line tests if any line but 
the last one generates output. So, if you want to print twice, make two separate test cells instead 
of a single cell with:

.. code-block:: python

    print(1)
    print(2)

**If a question has no solution cell provided**, the question will either be removed from the output 
notebook entirely if it has only hidden tests or will be replaced with an unprompted 
``Notebook.check`` cell that runs those tests. In either case, the test files are written, but this 
provides a way of defining additional test cases that do not have public versions. Note, however, 
that the lack of a ``Notebook.check`` cell for questions with only hidden tests means that the tests 
are run *at the end of execution*, and therefore are not robust to variable name collisions.


.. _otter_assign_python_seeding:

Intercell Seeding
+++++++++++++++++

Otter Assign maintains support for :ref:`intercell seeding <seeding>` by allowing seeds to be set 
in solution cells. To add a seed, write a line that ends with ``# SEED``; when Otter runs, this line 
will be removed from the student version of the notebook. This allows instructors to write code with 
deterministic output, with which hidden tests can be generated.

Note that seed cells are removed in student outputs, so any results in that notebook may be 
different from the provided tests. However, when grading, seeds are executed between each cell, so 
if you are using seeds, make sure to use **the same seed** every time to ensure that seeding before 
every cell won't affect your tests. You will also be required to set this seed as a configuration of 
the ``generate`` key of the assignment metadata if using Otter Generate with Otter Assign.


.. _otter_assign_python_manual_questions:

Manually Graded Questions
-------------------------

Otter Assign also supports manually-graded questions using a similar specification to the one 
described above. To indicate a manually-graded question, set ``manual: true`` in the question 
metadata. A manually-graded question is defined by three parts:

* a question cell with metadata
* (optionally) a prompt cell
* a solution cell

Manually-graded solution cells have two formats:

* If a code cell, they can be delimited by solution removal syntax as above.
* If a Markdown cell, the start of at least one line must match the regex 
  ``(<strong>|\*{2})solution:?(<\/strong>|\*{2})``.

The latter means that as long as one of the lines in the cell starts with ``SOLUTION`` (case 
insensitive, with or without a colon ``:``) in boldface, the cell is considered a solution cell. If 
there is a prompt cell for manually-graded questions (i.e. a cell between the question cell and 
solution cell), then this prompt is included in the output. If none is present, Otter Assign 
automatically adds a Markdown cell with the contents ``_Type your answer here, replacing this 
text._``.

Manually graded questions are automatically enclosed in ``<!-- BEGIN QUESTION -->`` and ``<!-- END 
QUESTION -->`` tags by Otter Assign so that only these questions are exported to the PDF when 
filtering is turned on (the default). In the autograder notebook, this includes the question cell, 
prompt cell, and solution cell. In the student notebook, this includes only the question and prompt 
cells. The ``<!-- END QUESTION -->`` tag is automatically inserted at the top of the next cell if it 
is a Markdown cell or in a new Markdown cell before the next cell if it is not.

An example of a manually-graded code question:

.. image:: images/assign_sample_code_manual.png
    :target: images/assign_sample_code_manual.png
    :alt: 


An example of a manually-graded written question (with no prompt):

.. image:: images/assign_sample_written_manual.png
    :target: images/assign_sample_written_manual.png
    :alt: 


An example of a manuall-graded written question with a custom prompt:

.. image:: images/assign_sample_written_manual_with_prompt.png
    :target: images/assign_sample_written_manual_with_prompt.png
    :alt: 


Ignoring Cells
--------------

For any cells that you don't want to be included in *either* of the output notebooks that are 
present in the master notebook, include a line at the top of the cell with the ``## Ignore ##`` 
comment (case insensitive) just like with test cells. Note that this also works for Markdown cells 
with the same syntax.

.. code-block:: python

    ## Ignore ##
    print("This cell won't appear in the output.")


Student-Facing Plugins
----------------------

Otter supports student-facing plugin events via the ``otter.Notebook.run_plugin`` method. To include 
a student-facing plugin call in the resulting versions of your master notebook, add a multiline 
plugin config string to a code cell of your choosing. The plugin config should be YAML-formatted as 
a mutliline comment-delimited string, similar to the solution and prompt blocks above. The comments 
``# BEGIN PLUGIN`` and ``# END PLUGIN`` should be used on the lines with the triple-quotes to delimit 
the YAML's boundaries. There is one required configuration: the plugin name, which should be a 
fully-qualified importable string that evaluates to a plugin that inherits from 
``otter.plugins.AbstractOtterPlugin``. 

There are two optional configurations: ``args`` and ``kwargs``. ``args`` should be a list of 
additional arguments to pass to the plugin. These will be left unquoted as-is, so you can pass 
variables in the notebook to the plugin just by listing them. ``kwargs`` should be a dictionary that 
mappins keyword argument names to values; thse will also be added to the call in ``key=value`` 
format.

Here is an example of plugin replacement in Otter Assign:

.. raw:: html

    <iframe src="../../_static/notebooks/html/assign-plugin.html"></iframe>
