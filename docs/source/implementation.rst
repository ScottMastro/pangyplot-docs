.. _implementation:
.. include:: substitutions.rst

Implementation Details
==================================

Back-end
------------------------------

Graph Structures: BubbleGun
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`BubbleGun 🔗 <https://github.com/fawaz-dabbaghieh/bubble_gun>`_ is a tool that identifies topological structures in a graph (GFA file).
These topological structures can be nested within each other, forming a hierarchical chain of superstructures.
The BubbleGun source code has been integrated into the |tool| back-end code.

Structure Definitions
~~~~~~~~~~~~~~~~~~~~~~~~

``Segment`` - a contiguous chunk of sequence with no variation. Basic nodes that make up a graph genome.

``SuperBubble`` - an acyclic, directed subgraph where nodes *s* and *t* are the source and sink nodes, respectively.
All paths that start at *s* end up at *t*.

``Bubble`` - a specific type of ``SuperBubble`` where there are only two disjoint paths between *s* and *t*. 
Ex. a simple bialleleic SNP or indel.

``BubbleChain`` - a subgraph containing one or more ``Bubble`` such that the sink node of one is the source node of the next.

The figure below is from the `BubbleGun publication 🔗 <https://doi.org/10.1093/bioinformatics/btac448>`_.

.. figure:: _images/bubblegun_figure.png
   :alt: figure from BubbleGun paper 
   :align: center

   A short ``BubbleChain`` composed of two simple SNP ``Bubble`` structures and one ``SuperBubble`` with another ``Bubble`` nested inside.

.. note:: 
    These definitions are used by BubbleGun. In |tool|, we refer to both types of bubbles simply as a |bubble|.
    We define subtypes |simple| and |super| to differentiate between a ``Bubble`` and a ``SuperBubble``, respectively.
    We also refer to a ``BubbleChain`` as simply |chain|.

Layout: odgi
^^^^^^^^^^^^^^^^^^^^^^^^

|tool| relies on `odgi 🔗 <https://github.com/pangenome/odgi>`_ to calculate the 2D layout of nodes.
The rGFA file is first converted into odgi format ``*.og``.

From the odgi documentation:

   The `odgi layout 🔗 <https://pangenome.github.io/odgi.github.io/rst/commands/odgi_layout.html>`_ command computes 2D layouts of the graph using stochastic gradient descent (SGD).
   The input graph must be sorted and id-compacted. The algorithm itself is described in 
   `Graph Drawing by Stochastic Gradient Descent 🔗 <https://arxiv.org/abs/1710.04626>`_. 
   The force-directed graph drawing algorithm minimizes the graph's energy function or stress level.

The command used to calculate the layout:

.. code-block:: bash

    odgi layout -i ${INPUT}.og -o ${OUTPUT}.lay -T ${OUTPUT}.lay.tsv

The ``*.lay.tsv`` output is structured as follows:

+-----+--------+-------------------+-----------+
| idx | X      | Y                 | component |
+=====+========+===================+===========+
| 0   | 1000   | 12547.3115187589  | 0         |
+-----+--------+-------------------+-----------+
| 1   | 165426 | 10586.0915549587  | 0         |
+-----+--------+-------------------+-----------+
| 2   | 165426 | 7320.81894996611  | 0         |
+-----+--------+-------------------+-----------+
| 3   | 165427 | 14814.159085348   | 0         |
+-----+--------+-------------------+-----------+
| 4   | 165427 | 14425.5419673736  | 0         |
+-----+--------+-------------------+-----------+
| 5   | 165445 | 15525.0135879779  | 0         |
+-----+--------+-------------------+-----------+
| 6   | 165445 | 12244.877453525   | 0         |
+-----+--------+-------------------+-----------+
| 7   | 165446 | 12979.6128977908  | 0         |
+-----+--------+-------------------+-----------+
| ... | ...    | ...               | ...       |
+-----+--------+-------------------+-----------+

For each ``S`` line in the rGFA file, two coordinate pairs are calculate, representing the coordinates for the start position and end position of each |segment|.
For example, for the first ``S`` line, the start position is given by line ``idx = 0`` and the end position by ``idx = 1``.


Database: Neo4j
^^^^^^^^^^^^^^^^^^^^^^^^

`Neo4j 🔗 <https://neo4j.com/>`_ is a graph database where data is stored as a series of Nodes and Relationships.
Neo4j mirrors the stucture of the graph genomes which makes it ideal for storage and queries. 
During setup of |tool|, an rGFA file is parsed and the ``S``, ``L``, ``P``, ``W`` lines are extracted.
|bubble| and |chain| superstructures are enumerated by BubbleGun.
Layout positions are calculated by odgi. These are all fed into the Neo4j database.

Schema
~~~~~~~~~~~~~~~~~~~~~

|segment| - The ``S`` lines from the rGFA file. They have these properties:

+--------------+---------+-----------------------------------------------------------------------------------+
| Property     | Type    | Description                                                                       |
+==============+=========+===================================================================================+
| **<id>**     | integer | Internal Neo4j ID.                                                                |
+--------------+---------+-----------------------------------------------------------------------------------+
| **id**       | integer | ID from rGFA file.                                                                |
+--------------+---------+-----------------------------------------------------------------------------------+
| **chrom**    | string  | Structured as ``genome_id#chromosome_id``. Optional.                              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **start**    | integer | Start position with respect to reference genome. Optional.                        |
+--------------+---------+-----------------------------------------------------------------------------------+
| **end**      | integer | End position with respect to reference genome. Optional.                          |
+--------------+---------+-----------------------------------------------------------------------------------+
| **sequence** | string  | DNA sequence, blank if node represents a deletion.                                |
+--------------+---------+-----------------------------------------------------------------------------------+
| **x1**       | float   | Layout x-coordinate for start position.                                           |
+--------------+---------+-----------------------------------------------------------------------------------+
| **y1**       | float   | Layout y-coordinate for start position.                                           |
+--------------+---------+-----------------------------------------------------------------------------------+
| **x2**       | float   | Layout x-coordinate for end position.                                             |
+--------------+---------+-----------------------------------------------------------------------------------+
| **y2**       | float   | Layout y-coordinate for end position.                                             |
+--------------+---------+-----------------------------------------------------------------------------------+

|bubble| - Identified by BubbleGun. They have these properties:

+--------------+---------+-----------------------------------------------------------------------------------+
| Property     | Type    | Description                                                                       |
+==============+=========+===================================================================================+
| **<id>**     | integer | Internal Neo4j ID.                                                                |
+--------------+---------+-----------------------------------------------------------------------------------+
| **id**       | integer | ID from BubbleGun.                                                                |
+--------------+---------+-----------------------------------------------------------------------------------+
| **subtype**  | string  | To define different subtypes, ex. |simple|, |super|, etc. Optional.               |
+--------------+---------+-----------------------------------------------------------------------------------+
| **n**        | integer | Number of nodes inside the structure.                                             |
+--------------+---------+-----------------------------------------------------------------------------------+
| **size**     | integer | Total sum of the sequence length of every node inside the structure.              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **chrom**    | string  | Structured as ``genome_id#chromosome_id``. Optional.                              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **start**    | integer | Min start position of every node inside the structure. Optional.                  |
+--------------+---------+-----------------------------------------------------------------------------------+
| **end**      | integer | Max end position of every node inside the structure. Optional.                    |
+--------------+---------+-----------------------------------------------------------------------------------+
| **x**        | float   | Layout x-coordinate, based on center of mass (ie. sequence length).               |
+--------------+---------+-----------------------------------------------------------------------------------+
| **y**        | float   | Layout y-coordinate, based on center of mass (ie. sequence length).               |
+--------------+---------+-----------------------------------------------------------------------------------+
| **x1**       | float   | Min x-coordinate of every node inside the structure.                              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **y1**       | float   | Min y-coordinate of every node inside the structure.                              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **x2**       | float   | Max x-coordinate of every node inside the structure.                              |
+--------------+---------+-----------------------------------------------------------------------------------+
| **y2**       | float   | Max y-coordinate of every node inside the structure.                              |
+--------------+---------+-----------------------------------------------------------------------------------+

|chain| - Identified by BubbleGun. They have the same properties as above.


Relationships
~~~~~~~~~~~~~~~~~~~~~

|linksto| : The ``L`` lines from the rGFA.

 |segment| — |linksto| → |segment|

|end| : The source or sink node of a |bubble| or |chain|.


 |segment| — |end| → |bubble|

 |segment| ← |end| — |bubble|

 |segment| — |end| → |chain|

 |segment| ← |end| — |chain|

|inside| : Nodes that are part of a larger superstructure.

 |segment| — |inside| → |bubble|

 |bubble| — |inside| → |chain|

 |segment| — |inside| → |chain|


|parent| : Specifically, |chain| that are part of a larger superstructure.

 |chain| — |parent| → |bubble|

 |chain| — |parent| → |chain|

Example
~~~~~~~~~~~~~~~~~~~~~

.. figure:: _images/neo4j_bubble.svg
   :alt: Neo4j database schema
   :align: center
   :width: 600px

   Showing |segment| nodes as orange and |bubble| nodes as red. 
   A series of substitution variants alongside a larger deletion variant.
   The node at the bottom is a "null" |segment| that represents a deletion allele.

.. figure:: _images/neo4j_bubblechain.svg
   :alt: Neo4j database schema
   :align: center
   :width: 600px

   Hiding any |segment| nodes that were inside a |bubble|. 
   Showing |chain| nodes as light brown.
   Two |chain| are formed, separated by the large deletion allele.

.. figure:: _images/neo4j_superbubble.svg
   :alt: Neo4j database schema
   :align: center
   :width: 600px

   Hiding any nodes that were inside a |chain|.
   The left |chain| is inside a larger |bubble|, formed by the deletion allele.




Front-end
------------------------------


Rendering: D3 Force Graph
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

|tool| uses the `D3 Force Graph library 🔗 <https://github.com/vasturiano/force-graph>`_ to draw the graph representation.

This library is able to render the graph given a set of nodes and links.
In |tool|, segments of sequence are represented by nodes. 
These segments can have variable length and it was important to capture the length information in the rendering of the graph.
|tool| splits up longer segments into a series of connected D3 nodes, using thickly drawn links to give the illusion of length. 

.. figure:: _images/graph_explain.svg
   :alt: How the graph is drawn
   :width: 500px
   :align: center

   How the stylized |segment| appear (top) versus how they structured from the perspective of the D3 engine (bottom).

For example, ``Segment 426`` and ``Segment 427`` represent a SNP, and are drawn with a single D3 node.
In contrast, ``Segment 428`` is 932 base pairs long and is drawn with 9 nodes, internally named ``428#0`` through ``428#8``.


#todo forces
#todo annotations