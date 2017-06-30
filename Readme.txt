LinLogLayout is a simple program for computing graph layouts
(positions of the nodes of a graph in two- or three-dimensional space)
and graph clusterings for visualization and knowledge discovery.
It reads a graph from a file, computes a layout and a clustering, writes
the layout and the clustering to a file, and displays them in a dialog.
LinLogLayout can be used to identify groups of densely connected nodes
in graphs, like groups of friends or collaborators in social networks,
related documents in hyperlink structures (e.g. web graphs),
cohesive subsystems in software models, etc.
With a change of a parameter in the <code>main</code> method,
it can also compute classical "nice" (i.e. readable) force-directed layouts.

The program is primarily intended as a demo of the use of its core layouter
and clusterer classes MinimizerBarnesHut, MinimizerClassic, and
OptimizerModularity.  While MinimizerBarnesHut is faster, MinimizerClassic
is simpler and not limited to a maximum of three dimensions.


Usage: java LinLogLayout <dim> <inputfile> <outputfile>
Computes a <dim>-dimensional layout and a clustering for the graph
in <inputfile>, writes the layout and the clustering into <outputfile>,
and displays (the first 2 dimensions of) the layout and the clustering.
<dim> must be 2 or 3.

Input file format:
Each line represents an edge and has the format:
<source> <target> <nonnegative real weight>
The weight is optional, the default value is 1.0.

Output file format:
<node> <x-coordinate> <y-coordinate> <z-coordinate (0.0 for 2D)> <cluster>


Example:
java -cp bin LinLogLayout 2 examples/WorldImport1999.el WorldImport1999.lc


LinLogLayout optimizes LinLog and related energy models to compute layouts,
and Newman and Girvan's Modularity to compute clusterings. For more information
about LinLog and Modularity, see

[1] A. Noack: "Energy Models for Graph Clustering",
Journal of Graph Algorithms and Applications, Vol. 11, no. 2, pp. 453-480, 2007.
http://jgaa.info/volume11.html
[2] M. E. J. Newman: "Analysis of weighted networks", Physical Review E 70,
056131, 2004.
[3] A. Noack. "Modularity Clustering is Force-Directed Layout",
Preprint arXiv:0807.4052, 2008. http://arxiv.org/abs/0807.4052

For further documentation of LinLogLayout,
see the wiki at http://code.google.com/p/linloglayout/.


If you find LinLogLayout useful, please recommend it and cite the above papers.
Suggestions, questions, and reports of applications are welcome.
Send them to Andreas Noack, an@informatik.tu-cottbus.de.


---------------------------LayoutParameters---------------------------

The constructors of the layouter classes MinimizerBarnesHut and MinimizerClassic have, besides the graph (network), three parameters controlling properties of the computed layouts: repuExponent, attrExponent, and gravFactor. These parameters correspond to the three types of forces (energies) simulated for computing the layouts: pairwise repulsion between different nodes, pairwise attraction between adjacent nodes (i.e., nodes connected by an edge), and gravitation of each node to the barycenter of the layout.

repuExponent is technically the exponent of the node distance in the repulsion energy. (As an exception, the value 0.0 corresponds to logarithmic repulsion.) It controls the uniformity of the node distances in the layout -- the smaller, the more uniform. Typical values are 0.0 (as in the LinLog energy model) or smaller.

attrExponent is technically the exponent of the node distance in the attraction energy. It controls the uniformity of the edge lengths in the layout -- the greater, the more uniform. Typical values are 1.0 (as in the LinLog energy model) or greater.

If the difference attrExponent-repuExponent is 1 or only slightly larger than 1 (as in the LinLog energy model), then the resulting layouts clearly show the density structure (community structure), by grouping densely connected nodes and separating sparsely connected nodes. In most classical energy models, the difference is much greater than 1, e.g. attrExponent==3.0 and repuExponent==0.0 in the classic Fruchterman-Reingold energy model. This distributes the nodes much more uniformly, and thus generally produces more readable layouts, which however reflect the density structure less clearly.

gravFactor is technically the factor of the gravitation energy. It controls how strongly each node is attracted to the barycenter of all nodes in the layout; for example, a value of 1.0 means that its attractions to the barycenter is about as strong as its total attraction to all other nodes (which is usually much too strong). Gravitation distorts the layout, and can be eliminated by setting the gravFactor to 0.0 -- but then, unconnected components of the graph fly away into infinity. If unsure whether the graph has unconnected components, try 0.1 or 0.05 first before decreasing to 0.005 or 0.0.

The parameters repuExponent and attrExponent are also mentioned in the articles * A. Noack. Energy Models for Graph Clustering, Journal of Graph Algorithms and Applications, 11(2):453-480, 2007. * A. Noack. Modularity Clustering is Force-Directed Layout, Preprint arXiv:0807.4052, 2008. Unfortunately, the naming is not consistent: In Section 3.6 of the earlier article, the parameter r of the r-PolyLog energy model corresponds to the attrExponent, the repuExponent is always 0.0. Section III.A of the latter article provides a fairly detailed discussion of the exponents; its parameters a and r of the (a,r)-energy model correspond to attrExponent-1 and repuExponent-1, respective

--------------------------layoutTips----------------------------------------


Use more iterations. The number of iterations is a parameter of the method minimizeEnergy of the classes MinimizerBarnesHut and MinimizerClassic (called in the main method of class LinLogLayout). While 100 iterations suffice for many graphs up to a few hundred nodes, larger or "difficult" graphs may require more iterations. Increasing the number of iterations is particularly helpful if the energy values printed by LinLogLayout still decrease notably during the last 5 percent of the iterations (e.g., from iteration 95 to iteration 100).
Reduce or eliminate gravitation. The gravitation factor is a parameter of the constructor of the classes MinimizerBarnesHut and MinimizerClassic (called in the main method of class LinLogLayout). Gravitation attracts all nodes to the barycenter of the layout. This prevents nodes from being moved to far from the remaining graph, but also distorts the layout. To reduce this distortion, set the gravitation factor to a very small value like 0.01 or 0.001. To eliminate the distortion entirely, set the gravitation factor to 0.0 -- but only if the graph is connected, otherwise the unconnected components will fly away into infinity (which will likely cause the minimizer to produce warnings like "The node distances in the layout are extremely nonuniform.").
Compute several layouts and choose the best. Because the layouts are initialized randomly, each run of LinLogLayout produces a different layout. Run LinLogLayout several times and choose the layout that has the best (smallest) energy after the final iteration. (LinLogLayout prints the current energy to the console at each iteration.)
Clusterings

Compute several clusterings and choose the best. Unlike the layouter, the clusterer of LinLogLayout is deterministic by default, i.e. produces the same clustering at each run (on the same graph). However, it can be randomized by commenting in a line marked with "randomize contraction" in the class OptimizerModularity. After commenting in this line, run LinLogLayout several times and choose the clustering with the best (largest) modularity.

-------------Troubleshooting---------------

I don't see any node in the visualization window.

Check the box "Show all names." Now node names should be visible; the nodes are right besides the names. If the nodes are very small, then the distances between the nodes in the computed layout are very large, probably because the graph has (almost) unconnected components. Increase the parameter gravFactor of the constructor MinimizerBarnesHut or MinimizerClassic, which is called in the main method of the class LinLogLayout, to 0.1 (or even higher, if necessary) to avoid that the unconnected components fly to far away. See LayoutParameters for details.

Some nodes are placed so closely together that I can hardly discern them.

Decrease the parameter repuExponent and/or increase the parameter attrExponent of the constructor MinimizerBarnesHut or MinimizerClassic, which is called in the main method of the class LinLogLayout. E.g., try the values -1.0 and 2.0 instead of the defaults 0.0 and 1.0. See LayoutParameters for details.

LinLogLayout prints "The node distances in the layout are extremely nonuniform."

This means that the largest node distances in the layout are much (by several orders of magnitude) larger than the smallest node distances, which makes it very difficult to find good energy minima. Possible causes are unconnected components combined with an insufficient gravitation factor (see the first problem above), nodes of weight 0.0 (in the standard setting, these are nodes whose edges have all weight 0.0), or extremely nonuniform graph density combined with aggressive layout parameters (see the previous problem). Especially for large graphs (> 1000 nodes), this message may appear a few times without indicating a serious problem.

According to the tool output, the repulsion exponent changes during the energy minimization process.

This is just an internal trick of the energy minimizer to produce better layouts, in particular to avoid bad local energy minima. For some parameter values, energy is very difficult to minimize, and the minimizer is prone to get stuck in bad local minima. In this case, the minimizer starts with "easier" parameter values that produce similar layouts, and then refines these layouts with the final (user-specified) parameter values.

I want to modify the source code but don't understand the algorithms.

Unfortunately, there is not detailed documentation of the algorithms. MinimizerBarnesHut is based on a force calculation algorithm by Barnes and Hut, described in * J. Barnes and P. Hut. A hierarchical O(N logN) force-calculation algorithm. Nature, 324:446–449, 1986. * A. J. Quigley and P. Eades. FADE: Graph drawing, clustering, and visual abstraction. In J. Marks, editor, Proceedings of the 8th International Symposium on Graph Drawing (GD 2000), LNCS 1984, pages 197–210, Berlin, 2001. Springer-Verlag.

Multi-level algorithms for Modularity clustering, of which OptimizerModularity is a simple example, are described in * A. Noack and R. Rotta. Multi-Level Algorithms for Modularity Clustering. Preprint arXiv:0812.4073, 2008.


