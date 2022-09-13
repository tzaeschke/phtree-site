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

There are currently implementations for Java and C++ available:

 - **C++**: by [myself](https://github.com/tzaeschke/phtree-cpp) (a fork of Improbable's implementation), by [Improbable](https://github.com/improbable-eng/phtree-cpp) by [mcxme](https://github.com/mcxme/phtree)
 - **Java**: by [myself](https://github.com/tzaeschke/phtree)

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

