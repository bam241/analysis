WARNING: This repository will be superseded by the cyan effort and is no longer 
actively developed.

This repository provides a location of analysis and data visualization tools for
*Cyclus* output.

Overview
--------

The tool suite is nominally broken into 4 modules.

* Utility - provides access tools to the *Cyclus* database
* Tables - provides table descriptions of the *Cyclus* database
* Queries - provides an interface for standard queries of the Cyclus database
* Helpers - provides helper functions to aggregate data and time series

Usage
-----

Connection to the Database
==========================

The analysis tools use SQLAlchemy's `Object Relational Mapping
(ORM) <http://docs.sqlalchemy.org/en/rel_0_8/orm/tutorial.html>`_ interface to
assist in making querying the *Cyclus* database easier. Querying the database
requires an active session, which can be accessed via:

.. code-block:: python

	from utility import loadSession
	dbPath = 'some_relative_path_to_the_database'
	session = loadSession(dbPath)

Querying the Database
=====================

Any query can be made on the database through the session, obtained above. A
number of standard queries, however, are supported in the queries module. Some
examples are provided below.

Simulation Time Information
+++++++++++++++++++++++++++

Cyclus simulations run using a month-based time step. The queries module
provides easy access to information about the beginning and ending time steps.

.. code-block:: python
	
	import queries as q

	# the month index from which the simulation begins
	startMonth = q.startMonth(session)

	# the month index on which the simulation ends
	endMonth = q.endMonth(session)
	
	# the number of months in the simulation
	# equivalent to endMonth - startMonth + 1
	nMonths = q.nMonths(session)

The *Cyclus* database can store multiple simulation runs. Accordingly, the above
method signatures can take an optional parameter (simID) which is 1 by default.

Facility Deployment
+++++++++++++++++++

Another normal query to be made about *Cyclus* simulation output is the facility
deployment at each time step. Let's say you have a facility type named
hwr_reactor in your simulation. In order to determine the deployment of
hwr_reactors at a given timestep, one can perform the following:

.. code-block:: python

	facility_type = "hwr_reactor"
	timestep = startMonth
	deployment = q.nFacs(session, facility_type, timestep)

Material Flows
++++++++++++++

Another metric important to most fuel cycle analyses is the amount of material
flowing into or out of certain types of facilities. In *Cyclus* each "type" of
material is denoted by its commodity name. Let's return to the hwr_reactor
example and assume that you've named your input fuel "hwr_fuel" and your output
fuel "hwr_used_fuel". You can get the amount of material entering or leaving all
hwr_reactors in the following manner:

.. code-block:: python

	# fuel flowing into reactors 
	commodity_type = "hwr_fuel" 
	direction = "in" 
	flow = q.materialFlow(session, facility_type, commodity_type, timestep, direction = direction)

	# fuel flowing out of reactors
	commodity_type = "hwr_used_fuel"
	direction = "out"
	flow = q.materialFlow(session, facility_type, commodity_type, timestep, direction = direction)

If it's preferable to aggregate material flow over some time period, one can
add the optional endTime parameter to sum the flows over the range of the
starting time and ending time.

.. code-block:: python
	
	# the sum of all fuel flowing into reactors over the life of the simulation
	commodity_type = "hwr_fuel" 
	direction = "in" 
	startTime = startMonth
	endTime = endMonth
	flow = q.materialFlow(session, facility_type, commodity_type, startTime, endTime = endTime, direction = direction)

Enrichment Parameters
+++++++++++++++++++++

Other standard queries include the amount of SWUs and natural uranium used by
enrichment facilities in a simulation. These queries follow the same normal form
used above.

.. code-block:: python

	# the amount of SWUs used during a simulation by enrichment facilities
	swu_used = q.SWU(session, startMonth, endTime = endMonth)

	# the amount of natural uranium used during a simulation by enrichment facilities
	natl_u_used = q.natlU(session, startMonth, endTime = endMonth)

Producing Fuel Cycle Metrics
============================

The most general case of fuel cycle analytics use is to develop graphs of fuel
cycle metrics. The curves module provides a number of methods to develop such
metrics. The helpers module also provides some quality-of-life methods to easily
get year-based indices. 

Let's say you want want to graph the deployment curve, fuel use, and used fuel
production for hwr_reactors in your simulation year-by-year and plot the
result. You could perform such an operation by:

.. code-block:: python

	import matplotlib.pyplot as plt
	import helpers as h
	import curves as c

	# get yearly values
	inYears = True
	
	# get year indicies based on month indicies
	year_indicies = h.yearIndicies(startMonth, endMonth)
	
	# get facility deployment and plot the result
	facility_type = "hwr_reactor"
	deployment = c.growthCurve(session, facility_type, startMonth, endMonth, inYears = inYears)
	plt.plot(year_indicies, deployment)
	plt.show()

	# get input fuel flow and plot the result
	commodity_type = "hwr_fuel"
	direction = "in"
	input = c.materialFlowCurve(session, facility_type, commodity_type, startMonth, endMonth, direction = direction, inYears = inYears)
	plt.plot(year_indicies, input)
	plt.show()

	# get output fuel flow and plot the result
	commodity_type = "hwr_used_fuel"
	direction = "out"
	output = c.materialFlowCurve(session, facility_type, commodity_type, startMonth, endMonth, direction = direction, inYears = inYears)
	plt.plot(year_indicies, output)
	plt.show()
	
