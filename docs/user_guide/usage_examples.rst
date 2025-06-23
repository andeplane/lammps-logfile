Comprehensive usage examples
===========================

This page collects several examples showing how to use the
:mod:`lammps_logfile` package for analysing LAMMPS log files.  The
examples expand on the material in :ref:`getting-started` and cover
common tasks you might perform when working with log data.

Basic reading
-------------

The :class:`lammps_logfile.File` class reads a log file and exposes the
thermo data as numpy arrays.  Simply pass the path to the log file on
construction:

.. code-block:: python

   from lammps_logfile import File

   log = File("path/to/log.lammps")
   time = log.get("Time")
   temperature = log.get("Temp")

Plotting multiple runs
----------------------

If your simulation contains several ``run`` statements the log file will
store the thermo output from each run separately.  You can access these
sections by specifying ``run_num`` when calling :meth:`File.get`:

.. code-block:: python

   import matplotlib.pyplot as plt

   log = File("path/to/log.lammps")
   for i in range(log.get_num_partial_logs()):
       t = log.get("Time", run_num=i)
       temp = log.get("Temp", run_num=i)
       plt.plot(t, temp, label=f"run {i}")

   plt.legend()
   plt.show()

Working with Pandas
-------------------

For more advanced analysis the log sections can be converted to a
:class:`pandas.DataFrame` using :meth:`File.to_dataframe`:

.. code-block:: python

   df = log.to_dataframe(run_num=0)
   print(df.head())

The resulting dataframe contains one column for each thermo property
available in the selected run.

Running averages
----------------

Fluctuating signals such as pressure or temperature can be smoothed using
the :func:`lammps_logfile.running_mean` helper function:

.. code-block:: python

   from lammps_logfile import running_mean

   raw_temp = log.get("Temp")
   smooth_temp = running_mean(raw_temp, 100)

   plt.plot(log.get("Time"), raw_temp, label="raw")
   plt.plot(log.get("Time"), smooth_temp, label="averaged")
   plt.legend()
   plt.show()

Command line tool
-----------------

Simple inspection of a log file can also be done through the
``lammps_logplotter`` command that is installed with the package.
Listing available columns::

   lammps_logplotter -c path/to/log.lammps

Plotting temperature versus time with a running average of 50 steps::

   lammps_logplotter -x Time -y Temp -a 50 path/to/log.lammps

Colour utilities
----------------

The :mod:`lammps_logfile.utils` module exposes a few helper functions
that are useful when plotting data from several simulations.  The
:func:`get_color_value` function maps a value to a colour from a Matplotlib
colour map:

.. code-block:: python

   from lammps_logfile.utils import get_color_value

   color = get_color_value(value=300, minValue=250, maxValue=400,
                           cmap='plasma')
   plt.plot(t, temp, color=color)

Similarly :func:`get_matlab_color` returns colours from MATLAB's standard
colour order which can be handy for consistent styling across plots.

