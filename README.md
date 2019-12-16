# MetagenomeScope
[![Build Status](https://travis-ci.org/marbl/MetagenomeScope.svg?branch=master)](https://travis-ci.org/marbl/MetagenomeScope) [![Code Coverage](https://codecov.io/gh/marbl/MetagenomeScope/branch/master/graph/badge.svg)](https://codecov.io/gh/marbl/MetagenomeScope)

![Screenshot of MetagenomeScope's standard mode, showing a region of a biofilm assembly graph](https://user-images.githubusercontent.com/4177727/46389776-f1d63780-c688-11e8-82ae-13d58d6f4738.png "Screenshot of MetagenomeScope's standard mode, showing a region of a biofilm assembly graph.")

## NOTE: This is a development branch of MetagenomeScope!
It's not quite at feature parity with the main branch (it'll actually just
crash if you try to run it right now due to some restructuring changes I've
been making to the codebase), so if you want to use MetagenomeScope in
the meantime I recommend using the main branch. Sorry for the inconvenience!

## Summary

MetagenomeScope is an interactive visualization tool designed for metagenomic
sequence assembly graphs. The tool aims to display a [hierarchical
layout](https://en.wikipedia.org/wiki/Layered_graph_drawing) of the input graph
while emphasizing the presence of small-scale details that can correspond to
interesting biological features in the data.

To this end, MetagenomeScope
highlights certain "structural patterns" of contigs in the graph,
splits the graph into its connected components (only displaying one connected
component at a time),
and uses [Graphviz](https://www.graphviz.org/)'
[`dot`](https://www.graphviz.org/pdf/dotguide.pdf) tool to hierarchically
lay out each connected component of the graph.

MetagenomeScope also contains a bunch of other features intended to simplify
exploratory analysis of assembly graphs, including tools for scaffold
visualization, path finishing, and (optionally)
[SPQR tree](https://en.wikipedia.org/wiki/SPQR_tree) decomposition of
biconnected components in the graph.

MetagenomeScope is composed of two main components:

### 1. Preprocessing Script

MetagenomeScope's **preprocessing script** (contained in the
`metagenomescope/` directory of this repository) is a mostly-Python script that
takes as input an assembly graph file and produces a SQLite .db file that can
be visualized in MetagenomeScope's viewer interface.

This preprocessing step takes care of structural pattern detection,
graph layout, and (optionally) SPQR tree generation.

Once you've installed MetagenomeScope, running the preprocessing script is as
simple as

```bash
mgsc -i [input assembly graph file] -o [output .db file prefix]
```

#### What types of assembly graphs can I use as input?

Currently, this supports
LastGraph ([Velvet](https://www.ebi.ac.uk/~zerbino/velvet/)),
GML ([MetaCarvel](https://github.com/marbl/MetaCarvel)), and
[GFA](https://gfa-spec.github.io/GFA-spec/) input files.
Support for SPAdes FASTG files is in development.

### 2. Viewer Interface

MetagenomeScope's **viewer interface** (contained in the `viewer/` directory
of this repository) is a client-side web application that reads a .db file
generated by MetagenomeScope's preprocessing script and draws the corresponding
graph using [Cytoscape.js](https://js.cytoscape.org/).

This interface includes various features for interacting with the graph and the
identified structural patterns within it.

You should be able to access the viewer interface from most modern web browsers
(mobile browsers also work, although using a desktop browser is generally
recommended). This can be done locally (if the viewer interface code is
downloaded on your computer, you can just open it up in a web browser) or over
HTTP/HTTPS (you can host an instance of the viewer interface on a server, or
try out a demo at [mgsc.umiacs.io](https://mgsc.umiacs.io/)).

### Why is the software structured that way?

The bifurcated nature of the tool lends it a few advantages that have proved
beneficial when analyzing large graphs:

- The user can save a .db file generated by the preprocessing script and
  visualize that file an arbitrary number of later times,
  without incurring the costs of layout, pattern detection, etc. twice
- The user can host the viewer interface and a number of .db files on
  a server, allowing many users to view graphs with the only costs incurred
  being those of rendering the graphs in question

Also, this is just how things came together during development :)

## Documentation

Documentation for MetagenomeScope --
**including [installation instructions for the preprocessing script](https://github.com/marbl/MetagenomeScope/wiki/Installation-Instructions)**
-- is available on its GitHub wiki,
located [here](https://github.com/marbl/MetagenomeScope/wiki).

## Demo

A demo of MetagenomeScope's viewer interface is available at
[mgsc.umiacs.io](https://mgsc.umiacs.io/).

- You can use the "Demo .db file" button to load sample assembly graph files that
  are already hosted with the demo.
- You can also use the "Upload .db file" button to view .db files generated by
  MetagenomeScope's preprocessing script. However, to ensure compatibility
  between your .db files and the viewer interface, it's probably best to use the
  version of the viewer interface bundled in the `viewer/` directory of this
  repository.

See [this page](https://github.com/marbl/MetagenomeScope/wiki/Customizing-Your-Own-Demo) on the wiki for instructions on customizing your own hosted version of MetagenomeScope's viewer interface.

## License

MetagenomeScope is licensed under the
[GNU GPL, version 3](https://www.gnu.org/copyleft/gpl.html).

License information for MetagenomeScope's dependencies is included in the root directory of this repository, in `DEPENDENCY_LICENSES.txt`. License copies for dependencies distributed/linked with MetagenomeScope -- when not included with their corresponding source code -- are available in the `dependency_licenses/` directory.

## Acknowledgements

See the [acknowledgements page](https://github.com/marbl/MetagenomeScope/wiki/Acknowledgements) on the wiki for a list of acknowledgements
for MetagenomeScope's codebase.

## Contact

MetagenomeScope was created by members of the [Pop Lab](https://sites.google.com/a/cs.umd.edu/poplab/) in the [Center for Bioinformatics and Computational Biology](https://cbcb.umd.edu/) at the [University of Maryland, College Park](https://umd.edu/).

Feel free to email `mfedarko (at) ucsd (dot) edu` with any questions, suggestions, comments, concerns, etc. regarding the tool. You can also open an [issue](https://github.com/marbl/MetagenomeScope/issues) in this repository, if you'd like.
