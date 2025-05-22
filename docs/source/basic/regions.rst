.. _regions:
.. include:: ../substitutions.rst

Choosing a Region
==================================

Selecting by Coordinate
--------------------------

|tool| uses a coordinate system based on one of the reference paths embedded in the sequence graph. The primary coordinate system is specified during setup — typically using a reference genome such as GRCh38 or CHM13.

.. figure:: ../_images/go_button.png
   :alt: coordinate section
   :align: center

   Coordinate section.

.. raw:: html

   <p>
      <code><i class="fas fa-crosshairs"></i></code> Manually enter or adjust the coordinate range.
      <br>
      <code><i class="fas fa-copy"></i></code> Copy the current coordinate range.
      <br>
      <code><i class="fas fa-plus-minus"></i></code> Set the number of flanking base pairs to include.
      <br>
      <code><i class="fas fa-minus"></i></code> Include flanking sequence upstream of the coordinate range.
      <br>
      <code><i class="fas fa-plus"></i></code> Include flanking sequence downstream of the coordinate range.
      <br>
      <code>Go <i class="fas fa-bolt-lightning"></i></code> Retrieve and display the specified coordinate range.
   </p>

   Each method below also fills the <i class="fas fa-crosshairs"></i> with a set of coordinates.</p>


Selecting by Chromosome
--------------------------

Range selection can be done by choosing a chromosome and highlighting a region.

.. figure:: ../_images/chromosome_selector.png
   :alt: chromosome selector
   :align: center

   Chromosome Selector.

By default, |tool| assumes a standard set of human chromosomes. Future versions of the software may accomodate different chromsomes for non-human species. Non-canonical chromosomes can be found in the ``Other`` selection box.
Only one chromosome can be selected at a time.

.. figure:: ../_images/locus_selector.png
   :alt: locus selector
   :align: center

   Locus Selector.

A cytoband display of the highlighted chromosome is rendered. Clicking and dragging along the chromosome will select a specific region. At this time, large regions may not be viewable due to the amount of data, but this limitation is a high priority for future improvments. 



Selecting by Gene
--------------------------

Gene annotations in GFF3 format, such as those provided by `GENCODE <https://www.gencodegenes.org/human/>`_, may be preloaded into |tool| and indexed for search.
The gene search bar shows these genes and allow for rapid selection of a particular region.

.. figure:: ../_images/gene_selector.png
   :alt: gene selector
   :align: center

   Select by Gene.

