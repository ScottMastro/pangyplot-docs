.. _principles:
.. include:: substitutions.rst

Core Concepts
==================================

Pangenome visualization is challenging due to the scale and complexity inherent in genomic graph data. Below we outline the core obstacles and how |tool| approaches them.

Managing Billions of Nodes and Edges
------------------------------------

Pangenome graphs contain massive volumes of data, often billions of segments and links. |tool| uses `Neo4j`_, an enterprise-level graph database engine, to store and query this data efficiently. Neo4j stores graph-structured data as *nodes* and *relationships*, rather than rows and columns. The model mirrors pangenome structures naturally.

Optimizing Layout
-----------------

Visualizing pangenome graphs requires organizing the graph into two dimensions.

- The initial x-coordinate and y-coordinate positions are computed via `odgi layout`_.
- Dynamic web-based graph rendering and physics are powered by `force-graph`_ (based on `D3.js`_).

Balancing Large and Small Variants
----------------------------------

Complex variation such as structural variants (SVs) coexists with smaller variants like SNPs and indels, and it can be difficult to visualize them together.

`BubbleGun`_ identifies "bubbles", "superbubbles", and "bubble chains" within GFA data. These bubble structures can be collapsed to simplify the graph and reduce clutter. Users can selectively "pop" open bubbles to explore fine-scale variation when needed. An implementation of this algorithm is included in |tool|.

Coordinate Systems and Annotations
----------------------------------

Genomic coordinate systems are essential for querying and aligning biological features. While pangenomic data does not inherently have a primary coordinate system, |tool| was developed under a design philosophy that requires one. When uploading a graph, users designate a primary path to serve as the coordinate reference system.

`odgi position`_ is used to establish the start and end positions of segments. Segments that are not part of the primary coordinate system are anchored to the nearest segment on the primary path.

With coordinates established, standard GFF3 annotation files can be imported to enable feature visualization and biological interpretation.

.. _Neo4j: https://neo4j.com/
.. _force-graph: https://github.com/vasturiano/force-graph
.. _D3.js: https://d3js.org/
.. _BubbleGun: https://github.com/fawaz-dabbaghieh/bubble_gun
.. _odgi layout: https://pangenome.github.io/odgi.github.io/rst/commands/odgi_layout.html
.. _odgi position: https://pangenome.github.io/odgi.github.io/rst/commands/odgi_position.html
