[![Build Status](https://travis-ci.org/lukeasrodgers/odb2graphml.svg?branch=master)](https://travis-ci.org/lukeasrodgers/odb2graphml)

This quickly-hacked-together node package converts the JSON exported by Orientdb into GraphML.

The goal is for the generated GraphML to be compatible with importing into Neo4j, via [neo4j-shell-tools](https://github.com/jexp/neo4j-shell-tools).

## Installation and usage

```
npm install -g odb2graphml
odb2graphml --help
```

By default, this tool will remove any edges that don't have corresponding vertices/nodes. Neo4j's import tool will fail if it
encounters edges that reference absent nodes. If, for some reason, you want to keep these edges, pass the `-k` or `--keep`
option to the tool. This scenario (missing nodes) is likely to occur when doing a non-locking export of an OrientDB database.

## Example

```
$ odb2graphml inputfile.json -v Vertex1Name,Vertex2Name -e Edge1Name,Edge2Name,Edge3Name
Success! Written to out.graphml
$ neo4j-shell
Unable to find any JVMs matching version "1.7".
Welcome to the Neo4j Shell! Enter 'help' for a list of commands
NOTE: Remote Neo4j graph database service 'shell' at port 1337

neo4j-sh (?)$ import-graphml -i /Users/luke/tmp/out.graphml -t
GraphML-Import file /Users/luke/tmp/out.graphml rel-type RELATED_TO batch-size 40000 use disk-cache false
finish after 122776 row(s)  0. 99%: nodes = 96642 rels = 26134 properties = 150905 time 7080 ms total 7080 ms
GraphML import created 122776 entities.
neo4j-sh (?)$
```

## Requirements

The code uses some ES6 features like template strings, so you will need a version of nodejs that supports those: at least v4.0.0.

## Notes

The code should mostly work in its current verison, though there are almost certain bugs and edge cases I've missed.

It uses [oboe.js](http://oboejs.com/) for streaming JSON parsing, the idea being that some export files may be very large and we don't want to load them all into memory at once.
Hence, it should be able to handle files of (more or less) arbitrarily large size.

Other notes:

* In order to ensure all nodes are output into graphml before edges (which is required by neo4j's import tool), we make
two passes through the input file; we could avoid this by using temporary files (also hacky) or in-memory write streams (which
undermines the benefits of using streams in the first place). This approach is about 2x slower, but seems the least hacky, though
more advanced knowledge of nodejs streams might provide a better solution.
