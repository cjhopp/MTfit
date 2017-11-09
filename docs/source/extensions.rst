*********************************
Extending mtfit
*********************************

mtfit has been written with the view that it is desirable to be able to easily extend the code. This is done using `entry points <https://pythonhosted.org/setuptools/pkg_resources.html#entry-points>`_ from the `setuptools <https://pythonhosted.org/setuptools>`_ module.

The entry points are:

.. only:: latex

    .. tabularcolumns:: |l|L|

+-------------------------------------------------+----------------------------------------------------+
|Entry Point                                      |Descriptions                                        |
+=================================================+====================================================+
|:ref:`mtfit.cmd_opts <entry_point-0>`            |Command line options                                |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.cmd_defaults <entry_point-1>`        |Default parameters for the command line options     |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.tests <entry_point-2>`               |Test functions for the extensions                   |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.pre_inversion <entry_point-3>`       |Function to be called with all kwargs before the    |
|                                                 |inversion object is initialised                     |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.post_inversion <entry_point-4>`      |Function to be called with all available kwargs     |
|                                                 |after the inversion has occurred                    |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.extensions <entry_point-5>`          |Functions that replaces the call to the inversion   |
|                                                 |using all the kwargs                                |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.parsers <entry_point-6>`             |Functions that return the data dictionary from an   |
|                                                 |input filename                                      |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.location_pdf_parsers <entry_point-7>`|Functions that return the location PDF samples      |
|                                                 |from an input filename                              |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.output_data_formats <entry_point-8>` |Functions that format the output data into a given  |
|                                                 |type, often linked to the output format             |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.output_formats <entry_point-9>`      |Functions that output the results from the          |
|                                                 |output_data_formats                                 |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.process_data_types <entry_point10>`  |Functions to convert input data into correct        |
|                                                 |format for new data types in forward model          |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.data_types <entry_point11>`          |Functions to evaluate the forward model for new     |
|                                                 |data types                                          |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.parallel_algorithms <entry_point12>` |Search algorithms that can be run (in parallel)     |
|                                                 |like monte carlo random sampling                    |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.directed_algorithms <entry_point13>` |Search algorithms that are dependent on the         |
|                                                 |previous value (e.g. McMC)                          |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.sampling <entry_point14>`            |Function that generates new moment tensor samples   |
|                                                 |in the Monte Carlo random sampling algorithm        |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.sampling_prior <entry_point15>`      |Function that calculates the prior either in the    |
|                                                 |McMC algorithm or the MC bayesian evidence          |
|                                                 |estimate                                            |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.sample_distribution <entry_point16>` |Function that generates random samples according    |
|                                                 |to some source model                                |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.plot <entry_point17>`                |Callable class for source plotting using            |
|                                                 |matplotlib                                          |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.plot_read <entry_point18>`           |Function that reads the data from a file for the    |
|                                                 |MTplot class                                        |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.documentation <entry_point19>`       |Installs the documentation for the extension        |
+-------------------------------------------------+----------------------------------------------------+
|:ref:`mtfit.source_code <entry_point20>`         |Installs the source code documentation for the      |
|                                                 |extension                                           |
+-------------------------------------------------+----------------------------------------------------+

These entry points can be accessed by adding some arguments to the :mod:`setuptools` module ``setup.py`` script::

     kwargs['entry_points']={'entry_point_name':['key = function']}

Where ``kwargs`` is the keyword dictionary passed to the :mod:`setuptools` :py:func:`setup` function, and the ``entry_point_name`` is the desired entry point in the other package.
The ``key`` is the description of the :py:func:`function`, used for selecting it in the code (this should be described by the package), and the :py:func:`function` is the desired function to be called when this key is selected.

The different usages for these entry points are described below.

:download:`extensions/scatangle.py <../../src/mtfit/extensions/scatangle.py>` is an example extension structure, although it would be necessary to make a ``setup.py`` file to install it.


.. _entry_point-0:

mtfit.cmd_opts
--------------------------

This entry point handles command line options for extensions that have been added. It is called when parsing the command line options, and should not conflict with the options described in :doc:`cli`.

The function is called as::

    parser_group,parser_check=cmd_opts(parser_group,argparse=[True/False],defaults)

Where the ``parser_group`` is the :mod:`argparse` or :mod:`optparse` parser group depending on if :mod:`argparse` is installed (Python version 2.7 or later), defaults are the command line defaults (with corresponding entry points :ref:`entry_point-1`), and ``parser_check`` is the function called to check/process the parsers results.

An example cmd_opts function is::

    def parser_check(parser,options,defaults):
        flags=[]
        if options['bin_scatangle']:
            if not options['location_pdf_file_path']:
              options['location_pdf_file_path']=glob.glob(options['data_file']+\
                    os.path.sep+'*'+options['angle_extension'])
            if not type(options['location_pdf_file_path'])==list:
              options['location_pdf_file_path']=[options['location_pdf_file_path']]
            #flags=['no_data_file_ok','no_location_update']
            flags=['no_location_update']
        return options,flags

    def cmd_opts(group,argparse=ARGPARSE,defaults=PARSER_DEFAULTS):
        if argparse:
            group.add_argument("--bin-scatangle","--binscatangle","--bin_scatangle", \
                action="store_true",default=defaults['bin_scatangle'], \
                help="Bin the scatangle file to reduce the number of samples \
                    [default=False]. --bin-size Sets the bin size parameter .", \
                dest="bin_scatangle")
            group.add_argument("--bin-size","--binsize","--bin_size",type=float, \
                default=defaults['bin_size'],help="Sets the scatangle bin size parameter \
                    [default="+str(defaults['bin_size'])+"].",dest="bin_scatangle_size")
        else:
            group.add_option("--bin-scatangle","--binscatangle","--bin_scatangle", \
                action="store_true",default=defaults['bin_scatangle'],help="Bin the \
                    scatangle file to reduce the number of samples [default=False]. \
                    --bin-size Sets the bin size parameter .",dest="bin_scatangle")
            group.add_option("--bin-size","--binsize","--bin_size",type=float, \
                default=defaults['bin_size'],help="Sets the scatangle bin size \
                    parameter [default="+str(defaults['bin_size'])+"].", \
                dest="bin_scatangle_size")
        return group,parser_check

This is taken from :download:`extensions/scatangle.py <../../src/mtfit/extensions/scatangle.py>`.

These command line options will be added to the options mtfit is called with so can then be parsed by other functions in the extension.

The command line options for an extension can be installed using :mod:`setuptools` by adding the ``mtfit.cmd_opts`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={'mtfit.cmd_opts':['extension = mymodule:cmd_opts']}
          ...)


.. _entry_point-1:

mtfit.cmd_defaults
--------------------------

This entry point handles the default values and types for the command line options described in :ref:`entry_point-0`. It is called when parsing the command line options.

The function is called as::

    plugin_defaults,plugin_default_types=cmd_defaults()

Where both are dicts, and should contain defaults for the :ref:`entry_point-0`, although they can also update the normal :doc:`cli` defaults and default types. Both dictionaries are used for updating the defaults from the default file (see :doc:`setup`).

An example cmd_defaults function is::

    PARSER_DEFAULTS={
              'bin_scatangle':False,
              'bin_size':1.0,
              }
    PARSER_DEFAULT_TYPES={'bin_scatangle':[bool],'bin_size':[float]}
    def cmd_defaults():
        return(PARSER_DEFAULTS,PARSER_DEFAULT_TYPES)

This is taken from :download:`extensions/scatangle.py <../../src/mtfit/extensions/scatangle.py>`.


The default command line options for an extension can be installed using :mod:`setuptools` by adding the ``mtfit.cmd_defaults`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={'mtfit.cmd_defaults':['extension = mymodule:cmd_defaults']}
          ...)


.. _entry_point-2:

mtfit.tests
--------------------------

This entry point is used for any extensions to add tests to the test suite, which can be run using ``mtfit --test`` on the command line, or as ``mtfit.run_tests()`` from within python.

The function is called as::

    test_suite,debug_test_suite,parser_test_function=tests()

Where ``test_suite`` is the :class:`unittest.TestSuite` containing the TestSuite, created as::

    tests=[]
    tests.append(unittest.TestLoader().loadTestsFromTestCase(__ExtensionTestCase))
    test_suite=unittest.TestSuite(tests)

from each :class:`unittest.TestCase`.``debug_test_suite`` is a single :class:`~unittest.TestSuite` containing the tests, added as::

    suite.addTests(unittest.TestLoader().loadTestsFromTestCase(__ExtensionTestCase))

from each :class:`unittest.TestCase`. ``parser_test_function`` is a single function to test the parser handling and checking.

An example of these functions is taken from :download:`extensions/scatangle.py <../../src/mtfit/extensions/scatangle.py>`::

    class __ScatangleTestCase(unittest.TestCase):
        def setUp(self):
            global _DEBUG
            self.__setattr__('existing_scatangle_files', glob.glob('*.scatangle'))
        def tearDown(self):
            for fname in glob.glob('*.scatangle'):
                if fname not in self.existing_scatangle_files:
                    try:
                        os.remove(fname)
                    except Exception:
                        print 'Cannot remove ',fname
            import gc
            try:
                os.remove('test.scatangle')
            except Exception:
                pass
            gc.collect()

        def station_angles(self):
            .
            .
            .
            .
        def test_parse_scatangle(self):
            open('test.scatangle','w').write(self.station_angles())
            A,B=parse_scatangle('test.scatangle')
            self.assertEqual(B,[504.7, 504.7])
            self.assertEqual(len(A),2)
            self.assertEqual(sorted(A[0].keys()),['Azimuth','Name','TakeOffAngle'])
            A,B=parse_scatangle('test.scatangle',bin_size=1)
            self.assertEqual(B,[1009.4])
            self.assertEqual(len(A),1)
            self.assertEqual(sorted(A[0].keys()),['Azimuth','Name','TakeOffAngle'])
            open('test.scatangle','w').write('\n'.join([self.station_angles() \
                    for i in range(40)]))
            global _CYTHON
            import time
            t0=time.time()
            A,B=parse_scatangle('test.scatangle',bin_size=1)
            print 'C',time.time()-t0
            t0=time.time()
            _CYTHON=False
            A,B=parse_scatangle('test.scatangle',bin_size=1)
            print 'NoC',time.time()-t0
            _CYTHON=True
            os.remove('test.scatangle')

    def parser_tests(self,_parser,defaults,argparse):
        print 'bin_scatangles --bin-scatangle and --bin-scatangle-size check'
        options,options_map=_parser(['Test.i'],test=True)
        self.assertTrue(options['bin_scatangle']==defaults['bin_scatangle'])
        self.assertEqual(options['bin_scatangle_size'],defaults['bin_size'])
        options,options_map=_parser(['--bin_scatangle'],test=True)
        self.assertTrue(options['bin_scatangle'])
        self.assertEqual(options['bin_scatangle_size'],defaults['bin_size'])
        options,options_map=_parser(['--bin_scatangle','--bin-size=2.0'],test=True)
        self.assertTrue(options['bin_scatangle'])
        self.assertEqual(options['bin_scatangle_size'],2.0)

    def _debug_test_suite():
        suite=unittest.TestSuite()
        suite.addTests(unittest.TestLoader().loadTestsFromTestCase(__ScatangleTestCase))
        return suite

    def _test_suite():
        tests=[]
        tests.append(unittest.TestLoader().loadTestsFromTestCase(__ScatangleTestCase))
        return unittest.TestSuite(tests)

    def tests():
        return(_test_suite(),_debug_test_suite(),parser_tests)

Where :func:`tests` is the entry point function.

A test suite for an extension can be installed using :mod:`setuptools` by adding the ``mtfit.tests`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={'mtfit.tests':['extension = mymodule:tests']}
          ...)

(N.B. the different test suites can be empty, and the ``parser_test_function`` can just be a pass function).



.. _entry_point-3:

mtfit.pre_inversion
--------------------------

This entry point provides an opportunity to call a function before the :class:`mtfit.inversion.Inversion` object is created (e.g. for some additional data processing).

The plugin is called as::

        kwargs=pre_inversion(**kwargs)

And can change the kwargs passed to the  :class:`~mtfit.inversion.Inversion` object to create it.

The function should just return the initial kwargs if the command line option to select it is not ``True``, otherwise it will always be called.

An pre_inversion function can be installed using :mod:`setuptools` by adding the ``mtfit.pre_inversion`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.pre_inversion':['my_fancy_function = mymodule:main_function'],
                'mtfit.cmd_opts':['extension = mymodule:cmd_opts'],
                'mtfit.cmd_defaults':['extension = mymodule:cmd_defaults']}
          ...)

Where the :ref:`mtfit.cmd_opts <entry_point-0>` and :ref:`mtfit.cmd_defaults <entry_point-1>` entry points  have been included.


.. _entry_point-4:

mtfit.post_inversion
--------------------------

This entry point provides an opportunity to call a function after the :class:`mtfit.inversion.Inversion` object is created (e.g. for some additional data processing).

The plugin is called as::

        post_inversion(**kwargs)

The function should just return nothing if the command line option to select it is not ``True``, otherwise it will always be called.

An post_inversion function can be installed using :mod:`setuptools` by adding the ``mtfit.post_inversion`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.post_inversion':['my_fancy_function = mymodule:main_function'],
                'mtfit.cmd_opts':['extension = mymodule:cmd_opts'],
                'mtfit.cmd_defaults':['extension = mymodule:cmd_defaults']}
          ...)

Where the :ref:`mtfit.cmd_opts <entry_point-0>` and :ref:`mtfit.cmd_defaults <entry_point-1>`  entry points have been included.


.. _entry_point-5:

mtfit.extensions
--------------------------

This entry point allows functions that can replace the main call to the :class:`mtfit.inversion.Inversion` object and to the :func:`mtfit.inversion.Inversion.forward()` function.

The plugin is called as::

        result=ext(**kwargs)
        if result !=1
            return result

Where kwargs are all the command line options that have been set.

If the result of the extension is ``1`` the program will not exit (this should be the case if the kwargs option to call the extension is not True), otherwise it exits.

N.B it is necessary for an extension to also have installed functions for the entry points:

    * :ref:`mtfit.cmd_opts <entry_point-0>`,
    * :ref:`mtfit.cmd_defaults <entry_point-1>`,

and the function should check if the appropriate option has been selected on the command line (if it doesn't it will always run).

An extension function can be installed using :mod:`setuptools` by adding the ``mtfit.extensions`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.extensions':['my_fancy_function = mymodule:main_function'],
                'mtfit.cmd_opts':['extension = mymodule:cmd_opts'],
                'mtfit.cmd_defaults':['extension = mymodule:cmd_defaults']}
          ...)

Where the :ref:`mtfit.cmd_opts <entry_point-0>` and :ref:`mtfit.cmd_defaults <entry_point-1>` entry points  have been included.



.. _entry_point-6:

mtfit.parsers
--------------------------

The :ref:`mtfit.parsers <entry_point-6>` entry point allows additional input file parsers to be added. The CSV parser is added using this in the ``setup.py`` script::

    kwargs['entry_points']={'mtfit.parsers':['.csv = mtfit.inversion:parse_csv']}

:mod:`mtfit` expects to call the plugin (if the data-file extension matches) as::

    data=plugin(filename)


A parser for a new file format can be installed using :mod:`setuptools` by adding the ``mtfit.parsers`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.parsers':
                    ['.my_format = mymodule.all_parsers:my_format_parser_function']
                }
          ...
          )

The parser is called using::

    data=my_new_format_parser_function(filename)

Where the ``filename`` is the data filename and ``data`` is the data dictionary (see :ref:`creating-data-dictionary-label`).

When a new parser is installed, the format (.my_new_format) will be called if it corresponds to the data-file extension. However if the extension doesn't match any of the parsers it will try all of them.

.. _entry_point-7:

mtfit.location_pdf_parsers
--------------------------

This entry point allows additional location :term:`PDF` file parsers to be added

:mod:`mtfit` expects to call the plugin (if the extension matches) as::

    location_samples,location_probability=plugin(filename,number_station_samples)

Where number_station_samples is the number of samples to use (i.e subsampling if there are more samples in the location :term:`PDF`).

A parser for a new format can be installed using  :mod:`setuptools` by adding the ``mtfit.location_pdf_parsers`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.location_pdf_parsers':
                    ['.my_format = mymodule.all_parsers:my_format_parser_function']
            }
          ...)

The parser is called using::

    location_samples,location_probability=my_format_parser_function(filename,
            number_location_samples)

Where the ``filename`` is the location :term:`PDF` filename and ``number_location_samples`` is the number of samples to use (i.e subsampling if there are more samples in the location :term:`PDF`).


The expected format for the location_samples and location_probability return values are::

    location_samples=[
        {'Name':['S01','S02',...],'Azimuth':np.matrix([[121.],[37.],...]),
            'TakeOffAngle':np.matrix([[88.],[12.],...])},
         {'Name':['S01','S02',...],'Azimuth':np.matrix([[120.],[36.],...]),
            'TakeOffAngle':np.matrix([[87.],[11.],...])}
        ]
    location_probability=[0.8,1.2,...]

These are then used in a :term:`Monte Carlo method` approach to include the location uncertainty in the inversion (see :doc:`bayes`).

When a new parser is installed, the format (.my_new_format) will be called if it corresponds to the data-file extension. However if the extension doesn't match any of the parsers it will try all of them.


.. _entry_point-8:

mtfit.output_data_formats
--------------------------

A parser for a new output data format can be installed using :mod:`setuptools` by adding the ``mtfit.output_data_formats`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.output_data_formats':
                    ['my_format = mymodule.all_parsers:my_output_data_function']
                }
          ...)

The parser is called using::

    output_data=my_output_data_function(event_data,self.inversion_options,
        output_data,location_samples,location_sample_multipliers,
        self.multiple_events,self._diagnostic_output,*args,**kwargs)

Where the ``event_data`` is the dictionary of event data, ``self.inversion_options`` are the inversion options set using the ``-i`` command line argument (see :doc:`cli`), the location_sample parameters are the :term:`PDF`
 samples described above, and the ``multiple_events`` and ``_diagnostic_output`` are corresponding boolean flags.

The format is set using the ``--resultsformat`` command line argument (see :doc:`cli`) or the ``results_format``  function argument when initialising the  :class:`~mtfit.inversion.Inversion` object.

The resulting ``output_data`` is normally expected to be either a dictionary to be passed to the output_format function to write to disk, or a pair of dictionaries (``list``). However it is passed straight through to the output file format function so it is possible to have a custom ``output_data`` object that is then dealt with in the output file formats function (see :ref:`entry_point-9`).
When a new parser is installed, the format (``my_format``) will be added to the possible result formats on the command line (``--resultsformat`` option in :doc:`cli`).


.. _entry_point-9:

mtfit.output_formats
--------------------------

mtfit has an entry point for the function that outputs the results to a specific file format.

The function outputs the results from the :ref:`output_data_formats function <entry_point-8>` and returns a string to be printed to the terminal and the output filename (it should change the extension as required) e.g.::

    out_string,filename=output_formatter(out_data,filename,JobPool,*args,**kwargs)

``JobPool`` is a :class:`mtfit.inversion.JobPool`, which handles job tasking if the inversion is being run in parallel. It can be passed a task (callable object) to write to disk in parallel.

The format is set using the ``--format`` command line argument (see :doc:`cli`) or the ``format``  function argument when initialising the  :class:`~mtfit.inversion.Inversion` object.

A new format can be installed using  :mod:`setuptools` by adding the ``mtfit.output_formats`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.output_formats':
                    ['my_format = mymodule.all_parsers:my_output_format_function']
                }
          ...)

The parser is called using::

    output_string,fname=my_output_format_function(output_data,
            fname,pool,*args,**kwargs)

Where the ``fname`` is the output filename and ``output_data`` is the output data from the output data parser (see :ref:entry_point-8`). ``pool`` is the :class:`mtfit.inversion.JobPool`.

When a new parser is installed, the format (``my_format``) will be added to the possible output formats on the command line (``--format`` option in :doc:`cli`).


.. _entry_point10:

mtfit.process_data_types
--------------------------

A function to process the data from the input data to the correct format for an :ref:`mtfit.data_types <entry_point11>` extension. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.process_data_types`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.process_data_types':
                    ['my_data_type = mymodule.all_parsers:my_data_type_preparation']
                }
          ...)

The function is called using::

    extension_data_dict=extension_function(event)

where event is the data dictionary (keys correspond to different data types and the settings of the inversion_options parameter).
The function returns a dict, with the station coefficients having keys ``a_***``  or ``aX_***`` where ``X`` is a single identifying digit. These station coefficients are a 3rd rank numpy array, with the middle index corresponding to the location samples.

.. _entry_point11:

mtfit.data_types
--------------------------

A function to evaluate the forward model likelihood for a new data-type. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.data_types`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.data_types':
                    ['my_data_type = mymodule.all_parsers:my_data_type_likelihood']
                }
          ...)

The inputs are prepared using an :ref:`mtfit.process_data_types <entry_point10>` extension.

The function is called using::

    ln_pdf=extension_function=(self.mt,**self.ext_data[key])

where ``self.ext_data[key]`` is the data prepared by the :ref:`mtfit.process_data_types <entry_point10>` function for this extension. The ``mt`` variable is a numpy array of moment tensor six vectors in the form::

    self.mt=np.array([[m11,...],
                      [m22,...],
                      [m33,...],
                      [sqrt(2)*m12,...],
                      [sqrt(2)*m13,...],
                      [sqrt(2)*m23,...]])

The station coefficients for the extension should be named as ``a_***`` or ``aX_***`` where ``X`` is a single identifying digit, and be a 3rd rank numpy array, with the middle index corresponding to the location samples.
The function returns a :class:`mtfit.probability.LnPDF` for the moment tensors provided. If the function does not exist, an error is raised, and the result ignored.

The function should handle any c/cython calling internally.

.. warning::

    It is assumed that the data used is independent, but this must be checked by the user.

Relative inversions can also be handled, but the extension name requires ``relative`` in it.

Relative functions are called using::

    ln_pdf,scale,scale_uncertainty=extension_function](self.mt,ext_data_1,ext_data_2)

Where ``ext_data_*`` is the extension data for each event as a dictionary. This dictionary, generated using the :ref:`mtfit.process_data_types <entry_point10>` function for this extension, should also contain a list of the receivers with observations, ordered in the same order as the numpy array of the data, as this is used for station indexing.

The ``scale`` and ``scale_uncertainty`` return variables correspond to estimates of the relative seismic moment between the two events, if it is generated by the extension function (if this is not estimated, ``1.`` and ``0.`` should be returned)


.. _entry_point12:

mtfit.parallel_algorithms
--------------------------

This extension provides an entry point for customising the search algorithm. This can be installed using :mod:`setuptools` by adding the ``mtfit.parallel_algorithms`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.parallel_algorithms':
                    ['my_new_algorithm = mymodule:my_new_algorithm_class']
                }
          ...)

The algorithm should inherit from :class:`mtfit.algorithms.monte_carlo_random._MonteCarloRandomSample`, or have the  functions :func:`initialise`, :func:`iterate`, :func:`__output__` and attributes :attr:`iteration`, :attr:`start_time`, and :attr:`pdf_sample` as a :class:`mtfit.sampling.Sample` or :class:`mtfit.sampling.FileSample` object.

The ``mtfit.parallel_algorithms`` entry point is for algorithms to replace the standard Monte Carlo random sampling algorithm, which can be called and run in parallel to generate new samples - see :func:`mtfit.inversion._random_sampling_forward`.

The algorithm is initialised as::

    algorithm=extension_algorithm(**kwargs)

where ``kwargs`` are the input arguments for the inversion object, and a few additional parameters such as the number of samples (``number_samples``), which is the number of samples per iteration, accounting for memory. Additional ``kwargs`` can be added using the :ref:`mtfit.cmd_opts<entry_point-0>` entry point.

The algorithm will be initialised, and expected to return the moment tensors to check in the forward model, and ``end=True``::

    mts,end=self.algorithm.initialise()

``end`` is a boolean flag to determine whether the end of the search has been reached, and mts is the numpy array of moment tensors in the form::

    mts=np.array([[m11,...],
                  [m22,...],
                  [m33,...],
                  [sqrt(2)*m12,...],
                  [sqrt(2)*m13,...],
                  [sqrt(2)*m23,...]])

After initialisation, the results are returned from the :class:`mtfit.inversion.ForwardTask` object as a dictionary which should be parsed using the :func:`iterate` function::

    mts,end=self.algorithm.iterate({'moment_tensors':mts,'ln_pdf':ln_p_total,'n':N})

The forward models can be run in parallel, either using :mod:`multiprocessing` or using MPI to pass the ``end`` flag. Consequently, these algorithms have no ordering, so can not depend on previous samples - to add an algorithm that is, it is necessary to use the :ref:`mtfit.directed_algorithms<entry_point13>` entry point.


.. _entry_point13:

mtfit.directed_algorithms
--------------------------

This extension provides an entry point for customising the search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.directed_algorithms`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.directed_algorithms':
                    ['my_new_algorithm = mymodule:my_new_algorithm_class']
                }
          ...)

The algorithm should inherit from :class:`mtfit.algorithms.__base__._BaseAlgorithm`, or have the functions :func:`initialise`, :func:`iterate`, :func:`__output__` and attribute :attr:`pdf_sample` as a :class:`mtfit.sampling.Sample` or :class:`mtfit.sampling.FileSample` object.

The ``mtfit.directed_algorithms`` entry point is for algorithms to replace the Markov chain Monte Carlo sampling algorithms - see :func:`mtfit.inversion._mcmc_sampling_forward`, using an :class:`mtfit.inversion.MCMCForwardTask` object

The algorithm is initialised as::

    algorithm=extension_algorithm(**kwargs)

where ``kwargs`` are the input arguments for the inversion object, and a few additional parameters such as the number of samples (``number_samples``), which is the number of samples per iteration, accounting for memory. Additional ``kwargs`` can be added using the :ref:`mtfit.cmd_opts<entry_point-0>` entry point.

The algorithm will be initialised, and expected to return the moment tensors to check in the forward model, and ``end=True``::

    mts,end=self.algorithm.initialise()

``end`` is a boolean flag to determine whether the end of the search has been reached, and ``mts`` is the numpy array of moment tensors in the form::

    mts=np.array([[m11,...],
                  [m22,...],
                  [m33,...],
                  [sqrt(2)*m12,...],
                  [sqrt(2)*m13,...],
                  [sqrt(2)*m23,...]])

After initialisation, the results are returned from the :class:`mtfit.inversion.ForwardTask` object as a dictionary which should be parsed using the iterate function::

    mts,end=self.algorithm.iterate({'moment_tensors':mts,'ln_pdf':ln_p_total,'n':N})

The forward models are run in order, so can depend on previous samples - to add an algorithm that does not need this, use the :ref:`mtfit.parallel_algorithms<entry_point12>` entry point.


.. _entry_point14:

mtfit.sampling
--------------------------

This extension provides an entry point for customising the moment tensor sampling used by the search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.sampling`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.sampling':
                    ['my_extension_name = mymodule:my_source_sampling']
                }
          ...)

The function should return a numpy array or matrix of normalised moment tensor six vectors in the form::

    mts=np.array([[m11,...],
                  [m22,...],
                  [m33,...],
                  [sqrt(2)*m12,...],
                  [sqrt(2)*m13,...],
                  [sqrt(2)*m23,...]])

If an alternate sampling is desired for the McMC case (ie. a different model), it is necessary to extend the algorithm class using the ``mtfit.directed_algorithms`` entry point.


.. _entry_point15:

mtfit.sampling_prior
--------------------------

This extension provides an entry point for customising the prior distribution of moment tensors used by the search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.sampling_prior`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.sampling_prior':
                    ['my_extension_name = mymodule:my_sampling_prior']
                }
          ...)

Different functions should be chosen for the Monte Carlo algorithms compared to the Markov chain Monte Carlo algorithms. In the Monte Carlo case, the prior is used to calculate the Bayesian evidence, and depends on the source type parameters.
It must reflect the prior distribution on the source samples as a Monte Carlo type integration is used to calculate it, and should return a float from two input floats::

    prior=prior_func(gamma,delta)

In the Markov chain Monte Carlo case, the function should return the prior of a sample, dependent on the selected model, again as a float. It is called as::

    prior=uniform_prior(xi,dc=None,basic_cdc=False,max_poisson=0,min_poisson=0)

where xi is a dictionary of the sample parameters e.g.::

    xi={'gamma':0.1,'delta':0.3,'kappa':pi/2,'h':0.5,'sigma':0}

If an alternate sampling is desired for the Markov chain Monte Carlo case (ie. a different model), it is necessary to extend the algorithm class using the ``mtfit.directed_algorithms`` entry point.


.. _entry_point16:

mtfit.sample_distribution
--------------------------

This extension provides an entry point for customising the source sampling used by the Monte Carlo search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.sample_distribution`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.sample_distribution':
                    ['my_extension_name = mymodule:my_random_model_func']
                }
          ...)

The model must generate a random sample according in the form of a numpy matrix or array::

    mts=np.array([[m11,...],
                  [m22,...],
                  [m33,...],
                  [sqrt(2)*m12,...],
                  [sqrt(2)*m13,...],
                  [sqrt(2)*m23,...]])

If an alternate sampling is desired for the Markov chain Monte Carlo case (ie. a different model), it is necessary to extend the algorithm class using the ``mtfit.directed_algorithms`` entry point.


.. _entry_point17:

mtfit.plot
--------------------------

This extension provides an entry point for customising the plot type the for mtfit.plot.MTplot object. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.plot`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.plot':
                    ['plottype = mymodule:my_plot_class']
                }
          ...)

The object should be a callable object which can accept the moment tensor 6-vector, matplotlib figure, matplotlib grid_spec and other arguments (see the :class:`mtfit.plot.plot_classes._BasePlot` class for an example), with the :func:`__call__` function corresponding to plotting the moment tensor.

The plottype name in the setup.py script should be lower case with no spaces, hypens or underscores (these are removed in parsing the plottype).



.. _entry_point18:

mtfit.plot_read
--------------------------

This extension provides an entry point for customising the input file parser for reading data for the mtfit.plot.MTplot object. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.plot_read`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.plot_read':
                    ['.file_extension = mymodule:my_read_function']
                }
          ...)

The function should accept an input filename and return a tuple of dicts for event and station data respectively



.. _entry_point19:

mtfit.documentation
--------------------------

This extension provides an entry point for customising the search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.documentation`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.documentation':
                    ['my_extension_name = mymodule:my_rst_docs']
                }
          ...)

The function should return a :ref:`ReST<http://docutils.sourceforge.net/rst.html>` string that can be written out when building the documentation using :mod:`sphinx`.

The name should be the extension name with _ replacing spaces. This will be capitalised into the link in the documentation.


.. _entry_point20:

mtfit.source_code
--------------------------

This extension provides an entry point for customising the search algorithm. This can be installed can be installed using :mod:`setuptools` by adding the ``mtfit.source_code`` entry point to the extension ``setup.py`` script::

    setup(...
          entry_points={
                'mtfit.source_code':
                    ['my_extension_name = mymodule:my_rst_source_code_docs']
                }
          ...)

The function should return a ``ReST`` string that can be written out when building the documentation using :mod:`sphinx`.

The name should be the extension name with _ replacing spaces. This will be capitalised into the link in the documentation.


