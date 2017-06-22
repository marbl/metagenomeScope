# MetagenomeScope

![Screenshot of MetagenomeScope's standard mode](https://user-images.githubusercontent.com/4177727/27416728-4c6297d8-56dd-11e7-9d89-472686c7a29e.png)

An interactive visualization tool designed for metagenomic sequence assembly
and scaffold graphs. The tool aims to prevent a semi-linearized,
hierarchical overview of the input graphs while emphasizing the
presence of certain structural patterns in these graphs.

To this end, MetagenomeScope highlights certain patterns of contigs in the
graph (bubbles, frayed ropes, chains, and "linear" cycles), splits graphs up by
their connected components, and uses [Graphviz](http://www.graphviz.org/)'
`dot` tool to hierarchically lay out each connected component of a graph.
MetagenomeScope also supports the use of
[SPQR tree](https://en.wikipedia.org/wiki/SPQR_tree) decompositions
(computed using [OGDF](http://www.ogdf.net/doku.php)) to present an
iteratively expandable hierarchical overview of a graph.

MetagenomeScope is composed of two main components:

1. The preprocessing script (contained in the `graph_collator/` directory of
   this repository), a Python and C++ script
   that takes as input an assembly or scaffold
   graph file and produces a SQLite .db file that can be visualized in the
   viewer interface. `collate.py` is the main script that needs to be run
   here; it uses `spqr.cpp` to interface with OGDF to generate SPQR tree
   decompositions.
   This preprocessing step takes care of
   graph layout, pattern detection, and SPQR tree generation.
   Currently, this supports LastGraph (Velvet), GML
   ([MetaCarvel](https://github.com/marbl/bambus3)), and GFA input
   files. Support for GFA2 and FASTG (SPAdes) files is planned.
   Please note that, although
   GFA files only have DNA sequences as an optional requirement
   (it's possible to denote the DNA of a sequence with just an asterisk),
   including DNA sequences in GFA files is currently required.

2. The viewer interface (contained in the `viewer/` directory of this
   repository), a client-side web application that reads a .db file
   generated by `collate.py` and renders the resulting graph using
   [Cytoscape.js](http://js.cytoscape.org/).
   This is coupled with an interface and "control panel" supporting various
   features to help with assembly finishing and exploratory analysis. (See
   the section below on
   [using the viewer interface](#running-the-viewer-interface) for an
   explanation of its current features.)

The bifurcated nature of the tool lends it a few advantages that have proved
beneficial when analyzing large graphs:

- The user can save a .db file and visualize the contents of the file
  an arbitrary number of later times, without incurring the costs of
  layout/pattern detection/etc. twice
- The user can host the viewer interface and a number of .db files on
  a server, allowing many users to view graphs with the only costs incurred
  being those of rendering the graphs in question

## System Requirements

### Preprocessing script

* Linux (in order to run the C++ binary contained in `graph_collator/spqr`) --
  working on getting a makefile set up to make this platform-independent
  eventually
* [Python 2.7](https://www.python.org/)
* [PyGraphviz](https://pygraphviz.github.io/)
* [Graphviz](http://graphviz.org/), with the `dot` and `sfdp` layout programs installed

### Viewer interface

* Any modern internet browser (smartphone/tablet mobile browsers should
  work, also) supported by Cytoscape.js
    * Google Chrome and Mozilla Firefox are recommended, but others should
      be fine. If you run into any problems using the viewer application on
      any browser, please let me know and I can look into it.

## Running the preprocessing script

`collate.py` is located in the graph\_collator folder. It can be
run from the command line;
see the [system requirements](#system-requirements) section above
for information on what software needs to be installed.

Running `collate.py` will process an assembly/scaffold graph file so that
it can be visualized. The syntax for this is

`./collate.py [-h] -i INPUTFILE -o OUTPUTPREFIX [-d OUTPUTDIRECTORY] [-pg]
    [-px] [-w] [-b BICOMPONENTSFILE]`

### Script output

The script will always produce a `.db` file that can be loaded in the viewer
application to visualize the assembly graph.

If the `-pg` argument is passed, `.gv` files (in the DOT language)
for each connected component of the assembly graph will be generated; if the
`-px` argument is passed, `.xdot` files (in the xdot language) for each
connected component of the assembly graph will be generated.

The script will also generate a few types of auxiliary files containing various
information about the structure of the assembly graph. These files are:

* `prefix_links`, where `prefix` is the output prefix passed via `-o`. Only one
  of these files will be generated per execution of `collate.py`. This file
  indicates all the edges in the assembly graph. If you pass in `-b` and the
  input assembly graph has unoriented contigs, then this file will not be
  generated (since it would be equivalent to the _single_links file in that
  case).
* `prefix_single_links`, where `prefix` is the output prefix passed via `-o`.
  This file will only be generated if the input assembly graph has unoriented
  contigs. In terms of currently supported input filetypes, this means that
  this file will only be generated when the input assembly graph is of type
  LastGraph or GFA.
* `prefix_bicmps`, where `prefix` is the output prefix passed via `-o`. Only
  one of these files will be generated per execution of `collate.py`. This
  file indicates the various separation pairs contained within the assembly
  graph (see [Nijkamp et al.](https://www.ncbi.nlm.nih.gov/pubmed/24058058)
  for a brief overview of separation pairs and their usage in bubble
  detection). It's possible to pass an existing version of this file using `-b`
  to the script, to prevent having to do the work of creating the file again.
* `component_D.info`, where `D` is an integer greater than 0. There will be one
  of these files created for every biconnected component contained within the
  assembly graph: these files indicate the contents of the SPQR tree defined
  for their corresponding biconnected component.
* `spqrD.gml`, where `D` is an integer greater than 0. These files correspond
  to `component_D.info` files: they indicate the connections between the
  metanodes of a SPQR tree.

The script requires all `component_D.info` and `spqrD.gml` files to be
removed from the output directory before it generates more of them.
If `-w` is enabled, then all existing files with corresponding names in the
output directory will be deleted; however, if `-w` is not enabled, then an
error will be raised.

Similarly, if files exist in the output directory with filenames overlapping 
those of the `prefix_links` and `prefix_bicmps` files, then those files will be
either deleted (if `-w` is enabled) or an error will be raised (if `-w` is not
enabled).

### Command-line argument descriptions

* `-i` The input assembly graph file to be used.
* `-o` The file prefix to be used for all files generated. As an example, given
  the argument
  `-o prefix`, the file `prefix.db` would be generated. If .gv and/or .xdot
  files are created (depending on the `-pg` or `-px` arguments, respectively),
  then those files will be numbered according to the relative size rank
  (in nodes) of their respective connected component within the assembly graph.
* `-d` This optional argument specifies the name of the directory in which
  all output files will be stored. If this argument is not indicated, then all
  files will be generated in the current working directory.
* `-pg` This optional argument produces DOT files (suffix .gv) in the output
  directory. As an example, given the arguments `-o prefix` and `-pg` for an
  assembly graph with 3 connected components, the files `prefix.db`,
  `prefix_1.gv`, `prefix_2.gv`, and `prefix_3.gv` would be created (where
  `prefix_1.gv` indicates the largest connected component by number of nodes,
  `prefix_2.gv` indicates the next largest connected component, and so on).
* `-px` This optional argument produces .xdot files in the output
  directory. These files are labelled in an identical fashion to `.gv` files,
  with the only difference in naming being the file suffix (.xdot instead of
  .gv).
* `-b` This optional argument lets you pass in an existing file indicating the
  separation pairs in the graph (to be used in the detection of complex
  bubbles) to the script.
* `-w` This optional argument allows the overwriting of output files
  (.db/.xdot/.gv/links/single_links/bicmps/.info/spqr.gml files).
  If this argument is **not** given, then:
    * An error will be raised if writing a .db file would cause another
      .db file to be overwritten.
    * A warning will be displayed if writing to a .gv or .xdot file would cause
      another .gv/.xdot file to be overwritten. In this case, the .gv/.xdot
      file in question simply would not be saved.
    * Note that the presence of files in the
      output directory that are conflicting-named folders (e.g. a
      directory named `e_coli.db/` in the output directory while attempting to
      produce a file named `e_coli.db`) will cause an error/warning to be
      raised regardless of whether or not `-w` is set.

## Running the viewer interface

You can load the interface in any modern browser. Chrome/Firefox are
recommended, but most modern browsers should be fine.

### Getting started

Database files generated by the preprocessing script that are stored on your
device locally can be loaded in the viewer interface using the
"Choose .db file" button. In the
[demo](http://www.cbcb.umd.edu/~mfedarko/MetagenomeScope/)
of MetagenomeScope, a number of sample database files are available for you to
try out using the "Demo .db" button.

Once you've loaded a file, the "Draw connected component" buttons can be used
to render a given connected component in the assembly/scaffold graph
represented by the database file in question. Note that connected components
are automatically sorted in descending order of number of nodes -- so connected
component 1 will be the largest, followed by 2, etc. In assembly
graphs, the number of connected components in the "standard mode" view of the
graph may be different from the number of connected components in the
"decomposition mode" view of the graph -- see the "Assembly info" button for
information on the number of connected components in each mode.

### Standard mode

This mode draws a normal view of the input graph, rendering each contig as
either one (if the input was a scaffold graph) or two (if the input was an
assembly graph) nodes. Edges between contigs are drawn accordingly.

In standard mode, structural patterns contained within the graph were
automatically highlighted and grouped together during layout. These patterns
can be easily identified by their background color and general structure.

### Decomposition mode

In these modes, each biconnected component within the input graph is collapsed
into the root node of its corresponding SPQR tree. These trees can then be
iteratively expanded via right-clicking (similar to how node groups are
collapsible/uncollapsible in standard mode).

In the "implicit" decomposition mode, expansions to the root node of a SPQR
tree show iteratively larger amounts of detail added to the root. In the
"explicit" decomposition mode, expansions actually result in the immediate
descendant metanodes in the SPQR tree being drawn, revealing more and more of
the literal SPQR tree structure for a given biconnected component.

Although "implicit" and "explicit" decompositions both show the same amount of
nodes and edges upon first being drawn, the layouts involving explicit
decomposition
will usually be messier (since more space needs to be allocated for the SPQR
tree structures) and will involve more nodes than their corresponding implicit
decomposition upon fully being uncollapsed.

## License

MetagenomeScope is licensed under the
[GNU GPL, version 3](https://www.gnu.org/copyleft/gpl.html).

## Contact

`mfedarko at umd dot edu`

## Acknowledgements

* The preprocessing script (in `collate.py`) uses
  [Graphviz](http://www.graphviz.org/)' `dot` and `sfdp` layout programs
  via [PyGraphviz](http://pygraphviz.github.io/).
* The preprocessing script (in `spqr.cpp`) uses
  [OGDF](http://www.ogdf.net/doku.php) to construct SPQR trees.
* Both the preprocessing script and the viewer interface use
  [sqlite3](https://sqlite.org/).
  In particular, the preprocessing script uses the built-in
  [Python sqlite3 module](https://github.com/ghaering/pysqlite)
  while the viewer interface uses [sql.js](https://github.com/kripken/sql.js/).
* The viewer interface uses [Cytoscape.js](https://js.cytoscape.org/) to render
  graphs on the client side.
    * Also, the toggling protocol used for the control panel of the
      viewer interface was inspired by a similar mechanism used in
      [this Cytoscape.js
      demo](http://js.cytoscape.org/demos/2ebdc40f1c2540de6cf0/).
* The viewer interface uses [jQuery](https://jquery.com/) and
  [Bootstrap](http://getbootstrap.com/) for various stylistic and functional
  purposes in the application.
    * The icons used to theme various controls in the viewer application are
      from the [Glyphicon](http://glyphicons.com/) Halflings set,
      included with Bootstrap.
