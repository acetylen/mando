Usage
=====

Defining commands
-----------------

A command is a function decorated with ``@command``. mando tries to extract as
much as information as possible from the function's docstring and its
signature.

The paragraph of the docstring is the command's **help**. For optimal results
it shouldn't be longer than one line. The second paragraph contains the
command's **description**, which can be as long as needed. If only one
paragraph is present, it is used for both the help and the description.
You can document the parameters with the common Sphinx's ``:param::`` syntax.

For example, this program generates the following helps::

    from mando import command, main


    @command
    def cmd(foo, bar):
        '''Here stands the help.

        And here the description of this useless command.

        :param foo: Well, the first arg.
        :param bar: Obviously the second arg. Nonsense.'''

        print(arg, bar)


    if __name__ == '__main__':
        main()

.. code-block:: console

    $ python command.py -h
    usage: command.py [-h] {cmd} ...

    positional arguments:
      {cmd}
        cmd       Here stands the help.

    optional arguments:
      -h, --help  show this help message and exit
    $ python command.py cmd -h
    usage: command.py cmd [-h] foo bar

    And here the description of this useless command.

    positional arguments:
      foo         Well, the first arg.
      bar         Obviously the second arg. Nonsense.

    optional arguments:
      -h, --help  show this help message and exit


Long and short options (flags)
------------------------------

You can specify short options in the docstring as well, with the ``:param:``
syntax. The recognized formats are these:

    * ``:param -O: Option help``
    * ``:param --option: Option help``
    * ``:param -o, --output: Option help``

Example::

    from mando import command, main


    @command
    def ex(foo, b=None, spam=None):
        '''Nothing interesting.

        :param foo: Bla bla.
        :param -b: A little flag.
        :param -s, --spam: Spam spam spam spam.'''

        print(foo, b, spam)

    if __name__ == '__main__':
        main()

Usage:

.. code-block:: console

    $ python short_options.py ex -h
    usage: short_options.py ex [-h] [-b B] [-s SPAM] foo

    Nothing interesting.

    positional arguments:
      foo                   Bla bla.

    optional arguments:
      -h, --help            show this help message and exit
      -b B                  A little flag.
      -s SPAM, --spam SPAM  Spam spam spam spam.
    $ python short_options.py ex 2
    ('2', None, None)
    $ python short_options.py ex 2 -b 8
    ('2', '8', None)
    $ python short_options.py ex 2 -b 8 -s 9
    ('2', '8', '9')
    $ python short_options.py ex 2 -b 8 --spam 9
    ('2', '8', '9')


How default arguments are handled
---------------------------------

If an argument has a default, then mando takes it as an optional argument,
while those which do not have a default are interpreted as positional
arguments. Here are the actions taken by mando when a default argument is
encountered:

+------------------------+-----------------------------------------------------+
| Default argument type  |   What mando specifies in ``add_argument()``        |
+========================+=====================================================+
| bool                   | *action* ``store_true`` or ``store_false`` is added |
+------------------------+-----------------------------------------------------+
| list                   | *action* ``append`` is added.                       |
+------------------------+-----------------------------------------------------+
| int                    | *type* ``int()`` is added.                          |
+------------------------+-----------------------------------------------------+
| float                  | *type* ``float()`` is added.                        |
+------------------------+-----------------------------------------------------+
| str                    | *type* ``str()`` is added.                          |
+------------------------+-----------------------------------------------------+

So, for example, if a default argument is an integer, mando will automatically
convert command line arguments to ``int()``::

    from mando import command, main


    @command
    def po(a=2, b=3):
        print(a ** b)


    if __name__ == '__main__':
        main()

.. code-block:: console

    $ python default_args.py po -h
    usage: default_args.py po [-h] [-a A] [-b B]

    optional arguments:
      -h, --help  show this help message and exit
      -a A
      -b B
    $ python default_args.py po -a 4 -b 9
    262144

Note that passing the arguments positionally does not work, because ``argparse``
expects optional args and ``a`` and ``b`` are already filled with defaults:

.. code-block:: console

    $ python default_args.py po
    8
    $ python default_args.py po 9 8
    usage: default_args.py [-h] {po} ...
    default_args.py: error: unrecognized arguments: 9 8

To overcome this, mando allows you to specify positional arguments' types in
the docstring, as explained in the next section.


Adding *type* and *metavar* in the docstring
--------------------------------------------

This is especially useful for positional arguments, but it is usually used for
all type of arguments.
The notation is this: ``:param {opt-name} <type>: Help``. ``<type>`` must be a
built-in type among the following:

    * ``<i>``, ``<int>``, ``<integer>`` to cast to ``int()``;
    * also ``<n>``, ``<num>``, ``<number>`` to cast to ``int()``;
    * ``<s>``, ``<str>``, ``<string>`` to cast to ``str()``;
    * ``<f>``, ``<float>`` to cast to ``float()``.

mando also adds ``<type>`` as a metavar.
Actual usage::

    from mando import command, main


    @command
    def pow(a, b, mod=None):
        '''Mimic Python's pow() function.

        :param a <float>: The base.
        :param b <float>: The exponent.
        :param -m, --mod <int>: Modulus.'''

        if mod is not None:
            print((a ** b) % mod)
        else:
            print(a ** b)


    if __name__ == '__main__':
        main()

.. code-block:: console

    $ python types.py pow -h
    usage: types.py pow [-h] [-m <int>] a b

    Mimic Python's pow() function.

    positional arguments:
    a                     The base.
    b                     The exponent.

    optional arguments:
    -h, --help            show this help message and exit
    -m <int>, --mod <int>
                        Modulus.
    $ python types.py pow 5 8
    390625.0
    $ python types.py pow 4.5 8.3
    264036.437449
    $ python types.py pow 5 8 -m 8
    1.0


Overriding arguments with ``@arg``
----------------------------------

TODO

Aliasing commands
-----------------

TODO