********************************
Composing Basic Queries
********************************

Lux provides a flexible language for communicating your analytical goals and intent to the system, so that the system can provide better and more relevant recommendations to you. In this tutorial, we will see various ways of specifying the intent, including the attributes and values that you are interested or not interested in, enumeration specifiers, as well as any constraints on the visualization encoding.

The primary way to set the intent is through :func:`lux.vis.LuxDataframe.set_intent`. :func:`lux.vis.LuxDataframe.set_intent` takes in a list of specification. We will first describe how intent can be specified through convenient shorthand descriptions, then we will describe advance usage via the :mod:`lux.vis.Clause` object.

Basic descriptions
------------------

Specifying attributes of interest
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can indicate interested in a single attribute, for instance `AverageCost`.

.. code-block:: python

    df.set_intent(['AverageCost'])

You might be interested in multiple attribute, for instance you might want to look at both `AverageCost` and `FundingModel`. When multiple items are specified, Lux puts all the specified items in the intent.

.. code-block:: python

    df.set_intent(['AverageCost','FundingModel'])

Let's say that in addition to `AverageCost`, you are interested in the looking at a list of attributes that are related to different financial measures, such as `Expenditure` or `MedianDebt`, and how they breakdown with respect to `FundingModel`. You can specify a list of desired attributes separated by the `|` symbol, which indicates an `OR` relationship. Lux automatically create combinations of the specified attributes. 

.. code-block:: python

    df.set_intent(["AverageCost|Expenditure|MedianDebt|MedianEarnings","FundingModel"])

Alternatively, you could also provide the specification as a list: 

.. code-block:: python

    df.set_intent([["AverageCost" 'Expenditure','MedianDebt' 'MedianEarnings'],"FundingModel"])

Specifying values of interest
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In Lux, you can also specify particular values corresponding to subsets of the data that you might be interested in. For example, you may be interested in only colleges in New England. 

.. code-block:: python

    df.set_intent(["MedianDebt","Region=New England"])

You can also specify multiple values of interest using the same `|` notation that we saw earlier. For example, you might be comparing colleges in New England, Southeast, and Far West.

.. code-block:: python

    df.set_intent(["MedianDebt","Region=New England|Southeast|Far West"])

.. note::
    You might be wondering what the difference is between specifying values of interest through the intent in Lux versus applying a filter directly on the dataframe through Pandas. By specifying the intent directly via Pandas, Lux is not be aware of what are the values of interest to users, so these values of interest will not be reflected in the recommendations.

    .. code-block:: python
        
        df[df["Region"]=="New England"]
    
    Specifying the values through the intent tells Lux that you care about colleges in the New England region. In this case, we see that Lux suggests visualizations in other `Region`s as recommendations.
    
    .. code-block:: python
        
        df.set_intent(["Region=New England"])

    So while both approaches applies the filter on the specified view, the slightly different interpretation results in different recommendations. In general, we encourage using Pandas for filtering if the user is certain about applying the filter (e.g., a cleaning operation deleting a specific data subset), and specify the intent in Lux if the user may want to experiment and change aspects related to the filter in their analysis. 

Advanced usage of :mod:`lux.vis.Clause`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The basic string-based descriptions provides a convenient way of specifying the intent. However, not all specification can be expressed through the descriptions, more complex specification can be expressed through the :mod:`lux.vis.Clause` object. The two modes of specification is essentially equivalent, with the :mod:`lux.compiler.Parser` parsing the specified string into the `description` field in the :mod:`lux.vis.Clause` object.

Specifying attributes or values of interest
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To see an example of how lux.Clause is used, we rewrite our earlier example of expressing interest in `AverageCost` as: 

.. code-block:: python
    
    df.set_intent([lux.Clause(attribute='AverageCost')])

Similarly, we can use :mod:`lux.vis.Clause` to specify values of interest:

.. code-block:: python 

    df.set_intent(['MedianDebt',
                    lux.Clause(attribute='Region',filter_op='=', value=['New England','Southeast','Far West']
                  ])

Both the `attribute` and `value` fields can take in either a single string or a list of attributes to specify items of interest. This example also demonstrates how we can intermix the `lux.Clause` specification alongside the basic string-based specification for convenience.

Adding constraints 
~~~~~~~~~~~~~~~~~~~

So far, we have seen examples of how to express existing use cases based on `lux.Clause`. Additional fields on the Clause object that acts as constraints to the specification. For example, we can indicate to Lux that we are interested in pinning `AverageCost` to the y axis.
    
.. code-block:: python
    
    df.set_intent([lux.Clause(attribute='AverageCost', channel='y')])

We can also set constraints on the type of aggregation that is used. For example, by default, we use `mean` as the default aggregation function for quantitative attributes.

.. code-block:: python

    df.set_intent(["HighestDegree","AverageCost"])

We can override the aggregation function to be `sum` instead. 

.. code-block:: python

    df.set_intent(["HighestDegree",lux.Clause("AverageCost",aggregation="sum")])

The possible aggregation values are the same as the ones supported in Pandas's `agg <https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.agg.html>`_ function, which can either be a string shorthand (e.g., "sum", "count", "min", "max", "median") or as a numpy aggregation function.

For example, we can change the aggregation function to be the point-to-point value (`np.ptp <https://numpy.org/doc/stable/reference/generated/numpy.ptp.html>`_) by inputting the numpy function.

.. code-block:: python

    df.set_intent(["HighestDegree",lux.Clause("AverageCost",aggregation=np.ptp)])

Specifying wildcards
~~~~~~~~~~~~~~~~~~~~~

Let's say that you are interested in *any* attribute with respect to `AverageCost`. Lux support *wildcards* (based on `CompassQL <https://idl.cs.washington.edu/papers/compassql/>`_ ), which specifies the enumeration of any possible attribute or values that satisfies the provided constraints.

.. code-block:: python
    
    df.set_intent(['AverageCost',lux.Clause('?')])

The space of enumeration can be narrowed based on constraints. For example, you might only be interested in looking at scatterplots of `AverageCost` with respect to quantitative attributes. 

.. code-block:: python
    
    df.set_intent(['AverageCost',lux.Clause('?',data_type='quantitative')])

The enumeration specifier can also be placed on the value field. For example, you might be interested in looking at how the distribution of `AverageCost` varies for all possible values of `Geography`.

.. code-block:: python
    
    df.set_intent(['AverageCost','Geography=?')])
or 

.. code-block:: python

    df.set_intent(['AverageCost',lux.Clause(attribute='Geography',filter_op='=',value='?')])