# The PH-tree

The PH-tree is a spatial index / multi-dimensional index. It is similar in function to other spatial indexes such as quadtrees, kd-trees or R-trees.

It supports the usual operations such as insert/remove, window queries and nearest neighbor queries. It can store points or axis-aligned boxes.

Compared to other spatial indexes the PH-tree's strengths are:

- Fast insert and remove operations. There is also no rebalancing, so insertion and removal times are quite predictable.
- Good scalability with dataset size. It is usually slower than other indexes when working on small datasets, such as < 100 or 1000 entries, but it scales very well with large datasets and has been tested with 100M entries.
- Good scalability with dimension. It works best between 3 and 10-20 dimensions. The Java versions has been tested with 1000 dimensions whwere it was about as fast as an R-Tree and faster than a kd-tree.
- It deals well with most types of datasets, e.g. it works fine with strongly clustered data. 
- Window queries are comparatively fast if they return a small result set, e.g. <10 entries. For larger result sets, other indexes are typically better.
- The PH-Tree is an *ordered* tree, i.e. when traversing the data, e.g. the results of a query, the data is [Morton-ordered (z-curve)](https://en.wikipedia.org/wiki/Z-order_curve).

Here are [results from performance tests](https://github.com/tzaeschke/TinSpin/blob/master/doc/benchmark-2017-01/Diagrams.pdf) with the [TinSpin](https://tinspin.org) framework on the PH-tree Java implementation.

# Implementations

PH-tree implementations that I am aware of:

 - **C++**: [here](https://github.com/tzaeschke/phtree-cpp) (my fork of Improbable's implementation), by [Improbable](https://github.com/improbable-eng/phtree-cpp) and by [mcxme](https://github.com/mcxme/phtree)
 - **Java**: [here](https://github.com/tzaeschke/phtree)

Other spatial indexes (Java) can be found in the [TinSpin index library](https://github.com/tzaeschke/tinspin-indexes).

There is also the [TinSpin](https://tinspin.org) spatial index testing framework.

## Support
Please contact me on [Discord](https://discord.gg/GNYjyyYq) or create GitHub Issues.


## Documents

- The PH-tree was developed at ETH Zurich and first published in
[The PH-Tree: A Space-Efficient Storage Structure and Multi-Dimensional Index](https://github.com/tzaeschke/phtree/blob/master/PH-Tree-v1.1-2014-06-28.pdf), 
Tilmann Zäschke, Christoph Zimmerli and Moira C. Norrie, 
Proceedings of Intl. Conf. on Management of Data (SIGMOD), 2014
- The current version of the PH-tree is discussed in more detail in this [The PH-Tree Revisited](https://github.com/tzaeschke/phtree/blob/master/PhTreeRevisited.pdf) (2015).
- There is a Master thesis about [Cluster-Computing and Parallelization for the Multi-Dimensional PH-Index](http://e-collection.library.ethz.ch/eserv/eth:47729/eth-47729-01.pdf) (2015).
- The hypercube navigation is discussed in detail in [Efficient Z-Ordered Traversal of Hypercube Indexes](https://github.com/tzaeschke/phtree/blob/master/Z-Ordered_Hypercube_Navigation.pdf) (2017).


# How does it work?

This explanation requires the reader to have a solid understanding of how [quadtrees](https://en.wikipedia.org/wiki/Quadtree)/[octrees](https://en.wikipedia.org/wiki/Octree) work (so please understand these first if you haven't already).
For the remainder of the discussion let's assume that we store 32 bit coordinates, however, storing 64 bit or other bit widths is straight forward.
Let's also assume that we use store 2D point data, however, other dimensions (up to 60 or more) and rectangle data is also possible.

The first difference to quadtrees is that a PH-tree stores only **integer coordinates**. Floating point coordinates can only be stored indirectly (a method for lossless conversion from floating point -> integer -> floating point is discussed below).

The second difference is that to **root node** is fixed with a **center point at (0,0) and an edge length of 2^31**. That means that the root node represent a square that encompasses all possible coordinates (assuming 32bit coordinates as stated above).

**TODO** Difference X is that in a PH-Tree, a child node does not have to be the same size as a quadrant in it's parent node. Instead, a child's size can be any fraction of the form child_length = parent_length/(2^x) as long as child_length >= 1. This is typically the case 


Like a quadtree, the PH-tree subdivides each node into quadrants of equals size, thus recusively dividing space. Using integer coordinates and a fixed large root node means that all quadrants have an edge length that is a power of two. The maximum depth of the tree is 32 (the root not has an edge length of 2^32, so at depth 32 we get edge length "2^32/2^32 = 1"). Also, it means that all node's center points and corner points are integer coordinates. This brings some advantages:

 * There is no room for problems caused by mathematical imprecision when dividing floating point coordinate repeatedly by 2.0.
 * Integer division by two is faster that floating point division. In fact, the edge length can be calculated using a right-bit-shift instead of division.
 * The maximum depth is 32. This is inherently limits degeneration in case of strongly clustered data.


DIfference X is that every node in a PH-Tree has at least two entries but most one entry per quadrant. Since a nodes has 2^dimension quadrants, every node can have up to 2^dimension entries. **For high dimensionality, nodes can get very large, but this is _not_  a problem!**

Difference X is that the PH-tree is essentially a *map* (whereas the quadtree is a multi-map) in the sense that for every coordinate, there can be only one entry. In order to act as a multi-map that can store multiple entries per coordinate, the PH-tree can store a *set* or *list* of entries for each coordinate. 

TODO
