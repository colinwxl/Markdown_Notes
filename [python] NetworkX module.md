## [NetworkX](https://networkx.github.io/) [python module]
2017-07-05
Release: 1.11

### Overview
NetworkX is a Python language software package for the creation, manipulation, and study of the structure, dynamics, and function of complex networks.

### Introduction
Classes are named using CamelCase (capital letters at the start of each word). functions, methods and variable names are lower_case_underscore (lowercase with an underscore representing a space between words).

#### NetworkX Basics
`import networkx as nx`

The following basic graph types are provided as Python classes:
- **Graph** This class implements an undirected graph. It ignores multiple edges between two nodes. It does allow self-loop edges between a node and itself.
- **DiGraph** Directed graphs, that is, graphs with directed edges. Operations common to directed graphs, (a subclass of Graph)
- **MultiGraph** A flexible graph class that allows multiple undirected edges between pairs of nodes. The additional flexibility leads to some degradation in performance, though usually not significant.
- **MultiDiGraph** A directed version of a MultiGraph.

```
G=nx.Graph()
G=nx.DiGraph()
G=nx.MultiGraph()
G=nx.MultiDiGraph()
```

All graph classes allow any hashable object as a node. Hashable objects include strings, tuples, integers, and more. Arbitrary edge attributes such as weights and labels can be associated with an edge.

The graph internal data structures are based on an adjacency list representation and implemented using Python dictionary datastructures. The graph adjacency structure is implemented as a Python dictionary of dictionaries; the outer dictionary is keyed by nodes to values that are themselves dictionaries keyed by neighboring node to the edge attributes associated with that edge. This “dict-of-dicts” structure allows fast addition, deletion, and lookup of nodes and neighbors in large graphs. The underlying datastructure is accessed directly by methods (the programming interface “API”) in the class definitions. All functions, on the other hand, manipulate graph-like objects solely via those API methods and not by acting directly on the datastructure. This design allows for possible replacement of the ‘dicts-of-dicts’-based datastructure with an alternative datastructure that implements the same methods.

#### Nodes and Edges
If the topology of the network is all you care about then using integers or strings as the nodes makes sense and you need not worry about edge data. If you have a data structure already in place to describe nodes you can simply use that structure as your nodes provided it is hashable. If it is not hashable you can use a unique identifier to represent the node and assign the data as a node attribute.

Edges often have data associated with them. Arbitrary data can associated with edges as an edge attribute. If the data is numeric and the intent is to represent a weighted graph then use the ‘weight’ keyword for the attribute. Some of the graph algorithms, such as Dijkstra’s shortest path algorithm, use this attribute name to get the weight for each edge.

Other attributes can be assigned to an edge by using keyword/value pairs when adding edges. You can use any keyword except ‘weight’ to name your attribute and can then easily query the edge data by that attribute keyword.

#### Graph Creation
NetworkX graph objects can be created in one of three ways:
- Graph generators – standard algorithms to create network topologies.
- Importing data from pre-existing (usually file) sources.
- Adding edges and nodes explicitly.

```
import networkx as nx
G=nx.Graph()
G.add_edge(1,2) # default edge data=1
G.add_edge(2,3,weight=0.9) # specify edge data

import math
G.add_edge('y','x',function=math.cos)
G.add_node(math.cos) # any hashable can be a node

elist=[('a','b',5.0),('b','c',3.0),('a','c',1.0),('c','d',7.3)]
G.add_weighted_edges_from(elist)
```
use Dijkstra’s algorithm to find the shortest weighted path:
```
G=nx.Graph()
e=[('a','b',0.3),('b','c',0.9),('a','c',0.5),('c','d',1.2)]
G.add_weighted_edges_from(e)
print(nx.dijkstra_path(G,'a','d'))
['a', 'c', 'd']

e = [('a','b'),('b','c'),('a','c'),('c','d')]
G.add_edges_from(e)
print(nx.shortest_path_length(G,'a','d'))
```
#### Data Structure:
NetworkX uses a “dictionary of dictionaries of dictionaries” as the basic network data structure. This allows fast lookup with reasonable storage for large sparse networks. The keys are nodes so G[u] returns an adjacency dictionary keyed by neighbor to the edge attribute dictionary. The expression G[u]\[v] returns the edge attribute dictionary itself. A dictionary of lists would have also been possible, but not allowed fast edge detection nor convenient storage of edge data.

Advantages of dict-of-dicts-of-dicts data structure:
- Find edges and remove edges with two dictionary look-ups.
- Prefer to “lists” because of fast lookup with sparse storage.
- Prefer to “sets” since data can be attached to edge.
- G[u][v] returns the edge attribute dictionary.
- n in G tests if node n is in graph G.
- for n in G: iterates through the graph.
- for nbr in G[n]: iterates through neighbors.

The data structure gets morphed slightly for each base graph class. For DiGraph two dict-of-dicts-of-dicts structures are provided, one for successors and one for predecessors. For MultiGraph/MultiDiGraph we use a dict-of-dicts-ofdicts-of-dicts where the third dictionary is keyed by an edge key identifier to the fourth dictionary which contains the edge attributes for that edge between the two nodes.

### Graph types
#### **Graph**(data=None, **attr)
- **data** (input graph) – Data to initialize graph. If data=None (default) an empty graph is created. The data can be an edge list, or any NetworkX graph object. If the corresponding optional Python packages are installed the data can also be a NumPy matrix or 2d ndarray, a SciPy sparse matrix, or a PyGraphviz graph.
- **attr** (keyword arguments, optional (default= no attributes)) – Attributes to add to graph as key=value pairs

If some edges connect nodes not yet in the graph, the nodes are added automatically. There are no errors when adding nodes or edges that already exist.
```
G = nx.Graph()
H = nx.Graph()
H.add_path([0,1,2,3,4,5,6,7,8,9])
G.add_nodes_from(H)
G.add_edges_from(H.edges())
len(G) # number of nodes in graph

for n,nbrsdict in G.adjacency_iter():
	for nbr,eattr in nbrsdict.items():
		if 'weight' in eattr:
			(n,nbr,eattr['weight'])
# more convenient:
G.edges(data='weight')
```
##### Subclasses(Advanced):
The Graph class uses a dict-of-dict-of-dict data structure. The outer dict (node_dict) holds adjacency lists keyed by node. The next dict (adjlist) represents the adjacency list and holds edge data keyed by neighbor. The inner dict (edge_attr) represents the edge data and holds edge attribute values keyed by attribute names.

Each of these three dicts can be replaced by a user defined dict-like object. In general, the dict-like features should be maintained but extra features can be added. To replace one of the dicts create a new graph class by changing the class(!) variable holding the factory for that dict-like structure. The variable names are node_dict_factory, adjlist_dict_factory and edge_attr_dict_factory.

**node_dict_factory** [function, (default: dict)] Factory function to be used to create the outer-most dict in the data structure that holds adjacency lists keyed by node. It should require no arguments and return a dict-like object.

**adjlist_dict_factory** [function, (default: dict)] Factory function to be used to create the adjacency list dict which holds edge data keyed by neighbor. It should require no arguments and return a dict-like object

**edge_attr_dict_factory** [function, (default: dict)] Factory function to be used to create the edge attribute dict which holds attrbute values keyed by attribute name. It should require no arguments and return a dict-like object.

Create a graph object that tracks the order nodes are added:
```
from collections import OrderedDict
class OrderedNodeGraph(nx.Graph):
	node_dict_factory=OrderedDict
G=OrderedNodeGraph()
G.add_nodes_from( (2,1) )
G.add_edges_from( ((2,2), (2,1), (1,1)) )
G.edges()
[Out]: [(2, 1), (2, 2), (1, 1)]
```
Create a graph object that tracks the order nodes are added and for each node track the order that neighbors are added:
```
class OrderedGraph(nx.Graph):
	node_dict_factory=OrderedDict
	adjlist_dict_factory = OrderedDict
G = OrderedGraph()
G.add_nodes_from( (2,1) )
G.add_edges_from( ((2,2), (2,1), (1,1)) )
G.edges()
[Out]: [(2, 2), (2, 1), (1, 1)]
```
Create a low memory graph class that effectively disallows edge attributes by using a single attribute dict for all edges. This reduces the memory used, but you lose edge attributes:
```
class ThinGraph(nx.Graph):
	all_edge_dict = {'weight': 1}
	def single_edge_dict(self):
		return self.all_edge_dict
	edge_attr_dict_factory = single_edge_dict
G = ThinGraph()
G.add_edge(2,1)
G.add_edge(2,2)
G[2][1] is G[2][2]
[Out]: True
```

##### Methods (Graph.):
1. Adding and removing nodes and edges
add_node(); add_nodes_from(); remove_node(); remove_nodes_from(); add_edge(); add_edges_from();
add_weighted_edges_from(); add_star(); add_path(); add_cycle(); clear()

2. Iterating over nodes and edges
nodes(); nodes_iter(); edges(); edges_iter(); get_edge_data(u,v[,default]); neighbors(n); neighbors_iter(n); adjacency_list(); adjacency_iter(); nbunch_iter()

3. Information about graph structure
has_node(n); has_edge(u,v); order() `return the number of nodes in the graph`; number_of_nodes(); degree(); degree_iter(); size() `return the number of edges`; number_of_edges(); nodes_with_selfloops(); selfloop_edges(); number_of_selfloops()

4. Making copies and subgraphs
copy(); to_undirected(); to directed(); subgraph(nbunch)

##### Methods (DiGraph./MultiGraph./MultiDiGraph):
...

### Algorithms
#### Approximation
##### Connectivity
In mathematics and computer science, connectivity is one of the basic concepts of graph theory: it asks for the minimum number of elements (nodes or edges) that need to be removed to disconnect the remaining nodes from each other. It is closely related to the theory of network flow problems. The connectivity of a graph is an important measure of its resilience as a network.
- all_pairs_node_connectivity(G[, nubunch, flow_func]) Compute node connectivity between all pairs of nodes
- local_node_connectivity(G, source, target[,...]) Compute node connectivity between source and target.
- node_connectivity(G[, s, t]) Return an approximation for node connectivity for a graph or digraph

```
# Platonic icosahedral graph has node connectivity 5 for each non adjacent node pair
from networkx.algorithms import approximation as approx
G = nx.icosahedral_graph()
approx.local_node_connectivity(G, 0, 6)
```

##### K-components
A 𝑘-component is a maximal subgraph of a graph G that has, at least, node connectivity 𝑘: we need to remove at least 𝑘 nodes to break it into more components.𝑘-components have an inherent hierarchical structure because they are nested in terms of connectivity: a connected graph can contain several 2-components, each of which can contain one or more 3-components, and so forth.
- k_components(G[, min_density=0.95]) Returns the approximate k-component structure of a graph G.
```
G = nx.petersen_graph()
k_components = approx.k_components(G)
```

##### Clique
A clique in an undirected graph G = (V, E) is a subset of the vertex set 𝐶 ⊆ 𝑉 , such that for every two vertices in C, there exists an edge connecting the two. This is equivalent to saying that the subgraph induced by C is complete (in some cases, the term clique may also refer to the subgraph).
- max_clique(G) Find the Maximum Clique
- clique_removal(G) Repeatedly remove cliques from the graph.

##### Clustering
- average_clusterig(G[, trials]) Estimates the average clustering coefficient of G.
The local clustering of each node in 𝐺 is the fraction of triangles that actually exist over all possible triangles in its neighborhood. The average clustering coefficient of a graph 𝐺 is the mean of local clusterings.

##### Dominating Set
A ‘dominating set‘ for an undirected graph *G with vertex set V and edge set E is a subset D of V such that every vertex not in D is adjacent to at least one member of D. An ‘edge dominating set‘ is a subset *F of E such that every edge not in F is incident to an endpoint of at least one edge in F.
- min_weighted_dominating_set(G[, weight]) Returns a dominating set that approximates the minimum weight node dominating set.
- min_edge_dominating_set(G) Return minimum cardinality edge dominating set.

##### Independent Set
Independent set or stable set is a set of vertices in a graph, no two of which are adjacent.
- maximum_independent_set(G) Return an approximate maximum independent set.

##### Matching
Given a graph G = (V,E), a matching M in G is a set of pairwise non-adjacent edges; that is, no two edges share a common vertex.
- min_maximal_matching(G) Returns the minimum maximal matching of G.

##### Ramsey
Ramsey numbers.
- ramsey_R2(G) Approximately computes the Ramsey number 𝑅(2; 𝑠, 𝑡) for graph.

##### Vertex Cover
Given an undirected graph 𝐺 = (𝑉,𝐸) and a function w assigning nonnegative weights to its vertices, find a minimum weight subset of V such that each edge in E is incident to at least one vertex in the subset.

#### Assortativity
##### Assortativity
Assortativity measures the similarity of connections in the graph with respect to the node degree.
- degree_assortativity_coefficient(G[, x, y, ...]) Compute degree assortativity of graph.
- attribute_assortativity_coefficient(G, attribute) Compute assortativity for node attributes.
- numeric_assortativity_coefficient(G, attribute) Compute assortativity for numerical node attributes.
- degree_pearson_correlation_coefficient(G[, ...]) Compute degree assortativity of graph.

##### Average neighbor degree
- average_neighbor_degree(G[, source, target, . . . ]) Returns the average degree of the neighborhood of each node.

##### Average degree connectivity
- average_degree_connectivity(G[, source, ...]) Compute the average degree connectivity of graph.
- k_nearest_neighbors(G[, source, target, ...]) Compute the average degree connectivity of graph.

##### Mixing
- attribute_mixing_matrix(G, attribute[, ...]) Return mixing matrix for attribute.
- degree_mixing_matrix(G[, x, y, weight, ...]) Return mixing matrix for attribute.
- degree_mixing_dict(G[, x, y, weight, nodes, ...]) Return dictionary representation of mixing matrix for degree.
- attribute_mixing_dict(G, attribute[, nodes, ...]) Return dictionary representation of mixing matrix for attribute.

#### Bipartite
Bipartite graphs 𝐵 = (𝑈, 𝑉,𝐸) have two node sets 𝑈, 𝑉 and edges in 𝐸 that only connect nodes from opposite sets. It is common in the literature to use an spatial analogy referring to the two node sets as top and bottom nodes.
`from networkx.algorithms import bipartite`

##### Basic functions
- is_bipartite(G) Returns True if graph G is bipartite, False if not.
- is_bipartite_node_set(G, nodes) Returns True if nodes and G/nodes are a bipartition of G.
- sets(G) Returns bipartite node sets of graph G.
- color(G) Returns a two-coloring of the graph.
- density(B, nodes) Return density of bipartite graph B.
- degrees(B, nodes[, weight]) Return the degrees of the two node sets in the bipartite graph B.

##### Matching
- eppstein_matching(G) Returns the maximum cardinality matching of the bipartite graph 𝐺.
- hopcroft_karp_matching(G) Returns the maximum cardinality matching of the bipartite graph 𝐺.
- to_vertex_cover(G, matching) Returns the minimum vertex cover corresponding to the given maximum matching of the bipartite graph 𝐺.

##### Matrix
- biadjacency_matrix(G, row_order[, ...]) Return the biadjacency matrix of the bipartite graph G.
- from_biadjacency_matrix(A[, create_using, ...]) Creates a new bipartite graph from a biadjacency matrix given as a SciPy sparse matrix.

##### Projections
- projected_graph(B, nodes[, multigraph]) Returns the projection of B onto one of its node sets.
- weighted_projected_graph(B, nodes[, ratio]) Returns a weighted projection of B onto one of its node sets.
- collaboration_weighted_projected_graph(B, nodes) Newman’s weighted projection of B onto one of its node sets.
- overlap_weighted_projected_graph(B, nodes[, ...]) Overlap weighted projection of B onto one of its node sets.
- generic_weighted_projected_graph(B,nodes[, ...]) Weighted projection of B with a user-specified weight function.

##### Spectral
- spectral_bipartivity(G[, nodes, weight]) Returns the spectral bipartivity.

##### Clustering
- clustering(G[, nodes, mode]) Compute a bipartite clustering coefficient for nodes.
- average_clustering(G[, nodes, mode]) Compute the average bipartite clustering coefficient.
- latapy_clustering(G[, nodes, mode]) Compute a bipartite clustering coefficient for nodes.
- robins_alexander_clustering(G) Compute the bipartite clustering of G.

##### Redundancy
- node_redundancy(G[, nodes]) Computes the node redundancy coefficients for the nodes in the bipartite graph G.

##### Centrality
- closeness_centrality(G, nodes[, normalized]) Compute the closeness centrality for nodes in a bipartite network.
- degree_centrality(G, nodes) Compute the degree centrality for nodes in a bipartite network.
- betweenness_centrality(G, nodes) Compute betweenness centrality for nodes in a bipartite network.

##### Generators
- complete_bipartite_graph(n1, n2[, create_using]) Return the complete bipartite graph 𝐾𝑛1,𝑛2.
- configuration_model(aseq, bseq[, ...]) Return a random bipartite graph from two given degree sequences.
- havel_hakimi_graph(aseq, bseq[, create_using]) Return a bipartite graph from two given degree sequences using a Havel-Hakimi style construction.
- reverse_havel_hakimi_graph(aseq, bseq[, ...]) Return a bipartite graph from two given degree sequences using a Havel-Hakimi style construction.
- alternating_havel_hakimi_graph(aseq, bseq[, ...]) Return a bipartite graph from two given degree sequences using an alternating Havel-Hakimi style construction.
- preferential_attachment_graph(aseq, p[, . . . ]) Create a bipartite graph with a preferential attachment model from a given single degree sequence.
- random_graph(n, m, p[, seed, directed]) Return a bipartite random graph.
- gnmk_random_graph(n, m, k[, seed, directed]) Return a random bipartite graph G_{n,m,k}.

#### Blockmodeling
- blockmodel(G, partitions[, multigraph]) Returns a reduced graph constructed using the generalized block modeling technique.

#### Boundary
- edge_boundary(G, nbunch1[, nbunch2]) Return the edge boundary.
- node_boundary(G, nbunch1[, nbunch2]) Return the node boundary.

#### Centrality
##### Degree
- degree_centrality(G) Compute the degree centrality for nodes.
- in_degree_centrality(G) Compute the in-degree centrality for nodes.
- out_degree_centrality(G) Compute the out-degree centrality for nodes.

##### Closeness
- closeness_centrality(G[, u, distance, ...]) Compute closeness centrality for nodes.

##### Betweenness
- betweenness_centrality(G[, k, normalized, ...]) Compute the shortest-path betweenness centrality for nodes.
- edge_betweenness_centrality(G[, k, ...]) Compute betweenness centrality for edges.

##### Current Flow Closeness
- current_flow_closeness_centrality(G[, ...]) Compute current-flow closeness centrality for nodes.

##### Current-Flow Betweenness
- current_flow_betweenness_centrality(G[, ...]) Compute current-flow betweenness centrality for nodes.
- edge_current_flow_betweenness_centrality(G) Compute current-flow betweenness centrality for edges.
- approximate_current_flow_betweenness_centrality(G) Compute the approximate current-flow betweenness centrality for nodes.

##### Eigenvector
- eigenvector_centrality(G[, max_iter, tol, ...]) Compute the eigenvector centrality for the graph G.
- eigenvector_centrality_numpy(G[, weight]) Compute the eigenvector centrality for the graph G.
- katz_centrality(G[, alpha, beta, max_iter, ...]) Compute the Katz centrality for the nodes of the graph G.
- katz_centrality_numpy(G[, alpha, beta, ...]) Compute the Katz centrality for the graph G.

##### Communicability
- communicability(G) Return communicability between all pairs of nodes in G.
- communicability_exp(G) Return communicability between all pairs of nodes in G.
- communicability_centrality(G) Return communicability centrality for each node in G.
- communicability_centrality_exp(G) Return the communicability centrality for each node of G
- communicability_betweenness_centrality(G[, ...]) Return communicability betweenness for all pairs of nodes in G.
- estrada_index(G) Return the Estrada index of a the graph G.

##### Dispersion
- dispersion(G[, u, v, normalized, alpha, b, c]) Calculate dispersion between 𝑢 and 𝑣 in 𝐺.

#### Chordal
A graph is chordal if every cycle of length at least 4 has a chord (an edge joining two nodes not adjacent in the cycle).
...

#### Clique
- enumerate_all_cliques(G) Returns all cliques in an undirected graph.
- find_cliques(G) Search for all maximal cliques in a graph.
- make_max_clique_graph(G[, create_using, name]) Create the maximal clique graph of a graph.
- make_clique_bipartite(G[, fpos, ...]) Create a bipartite clique graph from a graph G.
- graph_clique_number(G[, cliques]) Return the clique number (size of the largest clique) for G.

#### Clustering
- triangles(G[, nodes]) Compute the number of triangles.
- transitivity(G) Compute graph transitivity, the fraction of all possible triangles
present in G.
- clustering(G[, nodes, weight]) Compute the clustering coefficient for nodes.
- average_clustering(G[, nodes, weight, ...]) Compute the average clustering coefficient for the graph G.
- square_clustering(G[, nodes]) Compute the squares clustering coefficient for nodes.

#### Coloring
- greedy_color(G[, strategy, interchange]) Color a graph using various strategies of greedy graph coloring.

#### Communities
##### K-Clique
- k_clique_communities(G, k[, cliques]) Find k-clique communities in graph using the percolation method.

#### Components
##### Connectivity
- is_connected(G) Return True if the graph is connected, false otherwise.
- number_connected_components(G) Return the number of connected components.
- connected_components(G) Generate connected components.
- connected_component_subgraphs(G[, copy]) Generate connected components as subgraphs.
- node_connected_component(G, n) Return the nodes in the component of graph containing node n.

##### Strong connectivity
- is_strongly_connected(G) Test directed graph for strong connectivity.
- number_strongly_connected_components(G) Return number of strongly connected components in graph.
- strongly_connected_components(G) Generate nodes in strongly connected components of
graph.
- strongly_connected_component_subgraphs(G[, copy]) Generate strongly connected components as subgraphs.
- strongly_connected_components_recursive(G) Generate nodes in strongly connected components of graph.
- kosaraju_strongly_connected_components(G[, ...]) Generate nodes in strongly connected components of graph.
- condensation(G[, scc]) Returns the condensation of G.

##### Weak connectivity
- is_weakly_connected(G) Test directed graph for weak connectivity.
- number_weakly_connected_components(G) Return the number of weakly connected components in G.
- weakly_connected_components(G) Generate weakly connected components of G.
- weakly_connected_component_subgraphs(G[, copy]) Generate weakly connected components as subgraphs.

##### Attracting components
- is_attracting_component(G) Returns True if 𝐺 consists of a single attracting component.
- number_attracting_components(G) Returns the number of attracting components in 𝐺.
- attracting_components(G) Generates a list of attracting components in 𝐺.
- attracting_component_subgraphs(G[, copy]) Generates a list of attracting component subgraphs from 𝐺.

##### Biconnected components
- is_biconnected(G) Return True if the graph is biconnected, False otherwise.
- biconnected_components(G) Return a generator of sets of nodes, one set for each biconnected
- biconnected_component_edges(G) Return a generator of lists of edges, one list for each biconnected component of the input graph.
- biconnected_component_subgraphs(G[, copy]) Return a generator of graphs, one graph for each biconnected component of the input graph.
- articulation_points(G) Return a generator of articulation points, or cut vertices, of a graph.

#### Connectivity
##### K-node-componets
- k_components(G[, flow_func]) Returns the k-component structure of a graph G.

##### K-node-cutsets
- all_node_cuts(G[, k, flow_func]) Returns all minimum k cutsets of an undirected graph G.

##### Flow_based Connectivity
- average_node_connectivity(G[, flow_func]) Returns the average connectivity of a graph G.
- all_pairs_node_connectivity(G[, nbunch, ...]) Compute node connectivity between all pairs of nodes of G.
- edge_connectivity(G[, s, t, flow_func]) Returns the edge connectivity of the graph or digraph G.
- local_edge_connectivity(G, u, v[, ...]) Returns local edge connectivity for nodes s and t in G.
- local_node_connectivity(G, s, t[, ...]) Computes local node connectivity for nodes s and t.
- node_connectivity(G[, s, t, flow_func]) Returns node connectivity for a graph or digraph G.

##### Flow-based Minimum Cuts
- minimum_edge_cut(G[, s, t, flow_func]) Returns a set of edges of minimum cardinality that disconnects G.
- minimum_node_cut(G[, s, t, flow_func]) Returns a set of nodes of minimum cardinality that disconnects G.
- minimum_st_edge_cut(G, s, t[, flow_func, ...]) Returns the edges of the cut-set of a minimum (s, t)-cut.
- minimum_st_node_cut(G, s, t[, flow_func, ...]) Returns a set of nodes of minimum cardinality that disconnect source from target in G.

##### Stoer-Wagner minimum cut
- stoer_wagner(G[, weight, heap]) Returns the weighted minimum edge cut using the Stoer-Wagner algorithm.

##### Utils for flow-based connectivity
- build_auxiliary_edge_connectivity(G) Auxiliary digraph for computing flow based edge connectivity
- build_auxiliary_node_connectivity(G) Creates a directed graph D from an undirected graph G to compute flow based node connectivity.

#### Cores
- core_number(G) Return the core number for each vertex.
- k_core(G[, k, core_number]) Return the k-core of G.
- k_shell(G[, k, core_number]) Return the k-shell of G.
- k_crust(G[, k, core_number]) Return the k-crust of G.
- k_corona(G, k[, core_number]) Return the k-corona of G.

#### Cycles
- cycle_basis(G[, root]) Returns a list of cycles which form a basis for cycles of G.
- simple_cycles(G) Find simple cycles (elementary circuits) of a directed graph.
- find_cycle(G[, source, orientation]) Returns the edges of a cycle found via a directed, depth first traversal.

#### Directed Acyclic Graphs
- ancestors(G, source) Return all nodes having a path to 𝑠𝑜𝑢𝑟𝑐𝑒 in G.
- descendants(G, source) Return all nodes reachable from 𝑠𝑜𝑢𝑟𝑐𝑒 in G.
- topological_sort(G[, nbunch, reverse]) Return a list of nodes in topological sort order.
- topological_sort_recursive(G[, nbunch, reverse]) Return a list of nodes in topological sort order.
- is_directed_acyclic_graph(G) Return True if the graph G is a directed acyclic graph (DAG) or False if not.
- is_aperiodic(G) Return True if G is aperiodic.
- transitive_closure(G) Returns transitive closure of a directed graph
- antichains(G) Generates antichains from a DAG.
- dag_longest_path(G) Returns the longest path in a DAG
- dag_longest_path_length(G) Returns the longest path length in a DAG

#### Distance Measures
- center(G[, e]) Return the center of the graph G. `The center is the set of nodes with eccentricity equal to radius.`
- diameter(G[, e]) Return the diameter of the graph G. `The diameter(直径) is the maximum eccentricity.`
- eccentricity(G[, v, sp]) Return the eccentricity of nodes in G. `The eccentricity(偏心距) of a node v is the maximum distance from v to all other nodes in G.`
- periphery(G[, e]) Return the periphery of the graph G. `The periphery(外周) is the set of nodes with eccentricity equal to the diameter.`
- radius(G[, e]) Return the radius of the graph G. `The radius(半径) is the minimum eccentricity.`

#### Distance-Regular Graphs
A connected graph G is distance-regular if for any nodes x,y and any integers i,j=0,1,. . . ,d (where d is the graph diameter), the number of vertices at distance i from x and distance j from y depends only on i,j and the graph distance between x and y, independently of the choice of x and y.
- is_distance_regular(G) Returns True if the graph is distance regular, False otherwise.
- intersection_array(G) Returns the intersection array of a distance-regular graph.
- global_parameters(b, c) Return global parameters for a given intersection array.

#### Dominance
- immediate_dominators(G, start) Returns the immediate dominators of all nodes of a directed graph.
- dominance_frontiers(G, start) Returns the dominance frontiers of all nodes of a directed graph.

#### Dominating Sets
A dominating set for a graph 𝐺 = (𝑉,𝐸) is a node subset 𝐷 of 𝑉 such that every node not in 𝐷 is adjacent to at least one member of 𝐷.
- dominating_set(G[, start_with]) Finds a dominating set for the graph G.
- is_dominating_set(G, nbunch) Checks if nodes in nbunch are a dominating set for G.

#### Eulerian
An Eulerian graph is a graph with an Eulerian circuit.
- is_eulerian(G) Return True if G is an Eulerian graph, False otherwise.
- eulerian_circuit(G[, source]) Return the edges of an Eulerian circuit in G.

#### Flows
##### Maximum Flow
- maximum_flow(G, s, t[, capacity, flow_func]) Find a maximum single-commodity flow.
- maximum_flow_value(G, s, t[, capacity, . . . ]) Find the value of maximum single-commodity flow.
- minimum_cut(G, s, t[, capacity, flow_func]) Compute the value and the node partition of a minimum (s, t)-cut.
- minimum_cut_value(G, s, t[, capacity, flow_func]) Compute the value of a minimum (s, t)-cut.

##### Edmonds-Karp
- edmonds_karp(G, s, t[, capacity, residual, . . . ]) Find a maximum single-commodity flow using the Edmonds-Karp algorithm.

##### Shortest Augmenting Path
- shortest_augmenting_path(G, s, t[, . . . ]) Find a maximum single-commodity flow using the shortest augmenting path algorithm.

##### Preflow-Push
- preflow_push(G, s, t[, capacity, residual, . . . ]) Find a maximum single-commodity flow using the highestlabel preflow-push algorithm.

##### Utils
- build_residual_network(G, capacity) Build a residual network and initialize a zero flow.

##### Network Simplex
- network_simplex(G[, demand, capacity, weight]) Find a minimum cost flow satisfying all demands in digraph G.
- min_cost_flow_cost(G[, demand, capacity, weight]) Find the cost of a minimum cost flow satisfying all demands in digraph G.
- min_cost_flow(G[, demand, capacity, weight]) Return a minimum cost flow satisfying all demands in digraph G.
- cost_of_flow(G, flowDict[, weight]) Compute the cost of the flow given by flowDict on graph
G.
- max_flow_min_cost(G, s, t[, capacity, weight]) Return a maximum (s, t)-flow of minimum cost.

##### Capacity Scaling Minimum Cost Flow
- capacity_scaling(G[, demand, capacity, ...]) Find a minimum cost flow satisfying all demands in digraph G.

#### Graphical degree sequence
- is_graphical(sequence[, method]) Returns True if sequence is a valid degree sequence.
- is_digraphical(in_sequence, out_sequence) Returns True if some directed graph can realize the in- and out-degree sequences.
- is_multigraphical(sequence) Returns True if some multigraph can realize the sequence.
- is_pseudographical(sequence) Returns True if some pseudograph can realize the sequence.
- is_valid_degree_sequence_havel_hakimi(...) Returns True if deg_sequence can be realized by a simple graph.
- is_valid_degree_sequence_erdos_gallai(...) Returns True if deg_sequence can be realized by a simple graph.

#### Hieratchy
- flow_hierarchy(G[, weight]) Returns the flow hierarchy of a directed network.
Flow hierarchy is defined as the fraction of edges not participating in cycles in a directed graph.

#### Hybrid
A graph is locally (𝑘, 𝑙)-connected if for each edge (𝑢, 𝑣) in the graph there are at least 𝑙 edge-disjoint paths of length at most 𝑘 joining 𝑢 to 𝑣.
- kl_connected_subgraph(G, k, l[, low_memory, ...]) Returns the maximum locally (𝑘, 𝑙)-connected subgraph of G.
- is_kl_connected(G, k, l[, low_memory]) Returns True if and only if G is locally (𝑘, 𝑙)-connected.

#### Isolates
- is_isolate(G, n) Determine of node n is an isolate (degree zero).
- isolates(G) Return list of isolates in the graph.

#### Isomorphism(同构)
- is_isomorphic(G1, G2[, node_match, edge_match]) Returns True if the graphs G1 and G2 are isomorphic and False otherwise.
- could_be_isomorphic(G1, G2) Returns False if graphs are definitely not isomorphic.
- fast_could_be_isomorphic(G1, G2) Returns False if graphs are definitely not isomorphic.
- faster_could_be_isomorphic(G1, G2) Returns False if graphs are definitely not isomorphic.
##### Graph/DiGraph Matcher
...

#### Link Analysis
##### PageRank
PageRank computes a ranking of the nodes in the graph G based on the structure of the incoming links.
- pagerank(G[, alpha, personalization, ...]) Return the PageRank of the nodes in the graph.
- pagerank_numpy(G[, alpha, personalization, ...]) Return the PageRank of the nodes in the graph.
- pagerank_scipy(G[, alpha, personalization, ...]) Return the PageRank of the nodes in the graph.
- google_matrix(G[, alpha, personalization, ...]) Return the Google matrix of the graph.

##### Hits
The HITS algorithm computes two numbers for a node. Authorities estimates the node value based on the incoming links. Hubs estimates the node value based on outgoing links.
- hits(G[, max_iter, tol, nstart, normalized]) Return HITS hubs and authorities values for nodes.
- hits_numpy(G[, normalized]) Return HITS hubs and authorities values for nodes.
- hits_scipy(G[, max_iter, tol, normalized]) Return HITS hubs and authorities values for nodes.
- hub_matrix(G[, nodelist]) Return the HITS hub matrix.
- authority_matrix(G[, nodelist]) Return the HITS authority matrix.

#### Link Prediction
- resource_allocation_index(G[, ebunch]) Compute the resource allocation index of all node pairs in ebunch.
- jaccard_coefficient(G[, ebunch]) Compute the Jaccard coefficient of all node pairs in ebunch.
- adamic_adar_index(G[, ebunch]) Compute the Adamic-Adar index of all node pairs in
ebunch.
- preferential_attachment(G[, ebunch]) Compute the preferential attachment score of all node pairs in ebunch.
- cn_soundarajan_hopcroft(G[, ebunch, community]) Count the number of common neighbors of all node pairs in ebunch using community information.
- ra_index_soundarajan_hopcroft(G[, ebunch, ...]) Compute the resource allocation index of all node pairs in ebunch using community information.
- within_inter_cluster(G[, ebunch, delta, ...]) Compute the ratio of within- and inter-cluster common neighbors of all node pairs in ebunch.

#### Matching
- maximal_matching(G) Find a maximal cardinality matching in the graph.
- max_weight_matching(G[, maxcardinality]) Compute a maximum-weighted matching of G.

#### Minors
- contracted_edge(G, edge[, self_loops]) Returns the graph that results from contracting the specified edge.
- contracted_nodes(G, u, v[, self_loops]) Returns the graph that results from contracting u and v.
- identified_nodes(G, u, v[, self_loops]) Returns the graph that results from contracting u and v.
- quotient_graph(G, node_relation[, ...]) Returns the quotient graph of G under the specified equivalence relation on nodes.

#### Maximal independent set
- maximal_independent_set(G[, nodes]) Return a random maximal independent set guaranteed to contain a given set of nodes.

#### Minimum Spanning Tree(最小生成树)
A minimum spanning tree is a subgraph of the graph (a tree) with the minimum sum of edge weights.
- minimum_spanning_tree(G[, weight]) Return a minimum spanning tree or forest of an undirected weighted graph.
- minimum_spanning_edges(G[, weight, data]) Generate edges in a minimum spanning forest of an undirected weighted graph.

#### Operators
1. Unary operations
- complement(G[, name]) Return the graph complement of G.
- reverse(G[, copy]) Return the reverse directed graph of G.
2. Binary operations
- compose(G, H[, name]) Return a new graph of G composed with H.
- union(G, H[, rename, name]) Return the union of graphs G and H.
- disjoint_union(G, H) Return the disjoint union of graphs G and H.
- intersection(G, H) Return a new graph that contains only the edges that exist in both G and H.
- difference(G, H) Return a new graph that contains the edges that exist in G but not in H.
- symmetric_difference(G, H) Return new graph with edges that exist in either G or H but not both.
3. Operations on many graphs
- compose_all(graphs[, name]) Return the composition of all graphs.
- union_all(graphs[, rename, name]) Return the union of all graphs.
- disjoint_union_all(graphs) Return the disjoint union of all graphs.
- intersection_all(graphs) Return a new graph that contains only the edges that exist in all graphs.
4. Graph products
- cartesian_product(G, H) Return the Cartesian product of G and H.
- lexicographic_product(G, H) Return the lexicographic product of G and H.
- strong_product(G, H) Return the strong product of G and H.
- tensor_product(G, H) Return the tensor product of G and H.
- power(G, k) Returns the specified power of a graph.

#### Rich Club
- rich_club_coefficient(G[, normalized, Q]) Return the rich-club coefficient of the graph G.

#### Shortest Paths
- shortest_path(G[, source, target, weight]) Compute shortest paths in the graph.
If the source and target are both specified, return a single list of nodes in a shortest path from the source to the target.
If only the source is specified, return a dictionary keyed by targets with a list of nodes in a shortest path from the source to one of the targets.
If only the target is specified, return a dictionary keyed by sources with a list of nodes in a shortest path from one of the sources to the target.
If neither the source nor target are specified return a dictionary of dictionaries with path[source][target]=[list of nodes in path].
- all_shortest_paths(G, source, target[, weight]) Compute all shortest paths in the graph.
- shortest_path_length(G[, source, target, weight]) Compute shortest path lengths in the graph.
**weight** (None or string, optional (default = None)) – If None, every edge has weight/distance/cost 1. If a string, use this edge attribute as the edge weight. Any edge attribute not present defaults to 1.
- average_shortest_path_length(G[, weight]) Return the average shortest path length.
- has_path(G, source, target) Return True if G has a path from source to target, False otherwise.

##### Advanced Interface
1. Shortest path algorithms for unweighted graphs.
- single_source_shortest_path(G, source[, cutoff]) Compute shortest path between source and all other nodes reachable from source.
- single_source_shortest_path_length(G, source) Compute the shortest path lengths from source to all reachable nodes.
- all_pairs_shortest_path(G[, cutoff]) Compute shortest paths between all nodes.
- all_pairs_shortest_path_length(G[, cutoff]) Computes the shortest path lengths between all nodes in G.
- predecessor(G, source[, target, cutoff, ...]) Returns dictionary of predecessors for the path from source to all nodes in G.
```
G = nx.path_graph(4)
nx.predecessor(G,0)
[Out]: {0: [], 1: [0], 2: [1], 3: [2]}
```

2. Shortest path algorithms for weighed graphs.
- dijkstra_path(G, source, target, weight='weight') Returns the shortest path from source to target in a weighted graph G.
- dijkstra_path_length(G, source, target, weight='weight') Returns the shortest path length from source to target in a weighted graph.
- single_source_dijkstra_path(G, source, cutoff=None, weight= 'weight') Compute shortest path between source and all other reachable nodes for a weighted graph.
- single_source_dijkstra_path_length(G, source) Compute the shortest path length between source and all other reachable nodes for a weighted graph.
- all_pairs_dijkstra_path(G[, cutoff, weight]) Compute shortest paths between all nodes in a weighted graph.
- all_pairs_dijkstra_path_length(G[, cutoff, ...]) Compute shortest path lengths between all nodes in a weighted graph.
- single_source_dijkstra(G, source[, target, ...]) Compute shortest paths and lengths in a weighted graph G.
`length,path=nx.single_source_dijkstra(G,0)`
- bidirectional_dijkstra(G, source, target[, ...]) Dijkstra’s algorithm for shortest paths using bidirectional search.
```
G=nx.path_graph(5)
length,path=nx.bidirectional_dijkstra(G,0,4)
```
- dijkstra_predecessor_and_distance(G, source) Compute shortest path length and predecessors on shortest paths in weighted graphs.
- bellman_ford(G, source[, weight]) Compute shortest path lengths and predecessors on shortest
paths in weighted graphs.
Raises `NetworkXUnbounded` – If the (di)graph contains a negative cost (di)cycle, the algorithm raises an exception to indicate the presence of the negative cost (di)cycle. Note: any negative weight edge in an undirected graph is a negative cost cycle.
- negative_edge_cycle(G[, weight]) Return True if there exists a negative edge cycle anywhere in G.
- johnson(G[, weight]) Compute shortest paths between all nodes in a weighted graph using Johnson’s algorithm.
Johnson’s algorithm is suitable even for graphs with negative weights. It works by using the Bellman–Ford algorithm to compute a transformation of the input graph that removes all negative weights, allowing Dijkstra’s algorithm to be used on the transformed graph.

3. Floyd-Warshall algorithm for shortest paths. (Dense Graphs)
- floyd_warshall(G[, weight]) Find all-pairs shortest path lengths using Floyd’s algorithm.
- floyd_warshall_predecessor_and_distance(G[, ...]) Find all-pairs shortest path lengths using Floyd’s algorithm.
- floyd_warshall_numpy(G[, nodelist, weight]) Find all-pairs shortest path lengths using Floyd’s algorithm.

4. Shortest paths and path lengths using A* (“A star”) algorithm.
- astar_path(G, source, target[, heuristic, ...]) Return a list of nodes in a shortest path between source and target using the A* (“A-star”) algorithm.
- astar_path_length(G, source, target[, . . . ]) Return the length of the shortest path between source and target using the A* (“A-star”) algorithm.

#### Simple Paths
A simple path is a path with no repeated nodes.
- all_simple_paths(G, source, target[, cutoff]) Generate all simple paths in the graph G from source to target.
- shortest_simple_paths(G, source, target[, ...]) Generate all simple paths in the graph G from source to target, starting from shortest ones.

#### Swap
- double_edge_swap(G[, nswap, max_tries]) Swap two edges in the graph while keeping the node degrees fixed.
- connected_double_edge_swap(G[, nswap, ...]) Attempts the specified number of double-edge swaps in the graph G.

#### Traversal
##### Depth First Search
- dfs_edges(G[, source]) Produce edges in a depth-first-search (DFS).
- dfs_tree(G, source) Return oriented tree constructed from a depth-first-search from source.
- dfs_predecessors(G[, source]) Return dictionary of predecessors in depth-first-search from source.
- dfs_successors(G[, source]) Return dictionary of successors in depth-first-search from source.
- dfs_preorder_nodes(G[, source]) Produce nodes in a depth-first-search pre-ordering starting from source.
- dfs_postorder_nodes(G[, source]) Produce nodes in a depth-first-search post-ordering starting from source.
- dfs_labeled_edges(G[, source]) Produce edges in a depth-first-search (DFS) labeled by type.

##### Breadth First Search
- bfs_edges(G, source[, reverse]) Produce edges in a breadth-first-search starting at source.
- bfs_tree(G, source[, reverse]) Return an oriented tree constructed from of a breadth-firstsearch starting at source.
- bfs_predecessors(G, source) Return dictionary of predecessors in breadth-first-search from source.
- bfs_successors(G, source) Return dictionary of successors in breadth-first-search from source.

##### Depth First Search on Edges
- edge_dfs(G[, source, orientation]) A directed, depth-first traversal of edges in G, beginning at source.

#### Tree
##### Recognition
A forest is an acyclic, undirected graph, and a tree is a connected forest.
- is_tree(G) Returns True if G is a tree.
- is_forest(G) Returns True if G is a forest.
- is_arborescence(G) Returns True if G is an arborescence.
- is_branching(G) Returns True if G is a branching.

##### Branchings and Spanning Arborescences
- branching_weight(G[, attr, default]) Returns the total weight of a branching.
greedy_branching(G[, attr, default, kind]) Returns a branching obtained through a greedy algorithm.
- maximum_branching(G[, attr, default]) Returns a maximum branching from G.
- minimum_branching(G[, attr, default]) Returns a minimum branching from G.
- maximum_spanning_arborescence(G[, attr, default]) Returns a maximum spanning arborescence from G.
-minimum_spanning_arborescence(G[, attr, default]) Returns a minimum spanning arborescence from G.
- Edmonds(G[, seed]) Edmonds algorithm for finding optimal branchings and spanning arborescences.
Methods: find_optimum([attr, default, kind, style]) Returns a branching from G.

#### Triads
triadic_census(G) Determines the triadic census of a directed graph.

#### Vitality
closeness_vitality(G[, weight]) Compute closeness vitality for nodes.

### Function
#### Graph
- degree(G[, nbunch, weight]) Return degree of single node or of nbunch of nodes.
- degree_histogram(G) Return a list of the frequency of each degree value.
- density(G) Return the density of a graph.
undirected graphs: d = 2m/n(n-1); directed graphs: d = m/n(n-1)
where 𝑛 is the number of nodes and 𝑚 is the number of edges in 𝐺.
- info(G[, n]) Print short summary of information for the graph G or the node n.
- create_empty_copy(G[, with_nodes]) Return a copy of the graph G with all of the edges removed.
- is_directed(G) Return True if graph is directed.

#### Nodes
- nodes(G) Return a copy of the graph nodes in a list.
- number_of_nodes(G) Return the number of nodes in the graph.
- nodes_iter(G) Return an iterator over the graph nodes.
- all_neighbors(graph, node) Returns all of the neighbors of a node in the graph.
- non_neighbors(graph, node) Returns the non-neighbors of the node in the graph.
- common_neighbors(G, u, v) Return the common neighbors of two nodes in a graph.

#### Edges
- edges(G[, nbunch]) Return list of edges incident to nodes in nbunch.
- number_of_edges(G) Return the number of edges in the graph.
- edges_iter(G[, nbunch]) Return iterator over edges incident to nodes in nbunch.
- non_edges(graph) Returns the non-existent edges in the graph.

#### Attributes
- set_node_attributes(G, name, values) Set node attributes from dictionary of nodes and values
- get_node_attributes(G, name) Get node attributes from graph
- set_edge_attributes(G, name, values) Set edge attributes from dictionary of edge tuples and values.
- get_edge_attributes(G, name) Get edge attributes from graph

#### Freezing graph structure
- freeze(G) Modify graph to prevent further change by adding or removing nodes or edges.
- is_frozen(G) Return True if graph is frozen.

### Graph generators
#### Atlas
- graph_atlas_g() Return the list [G0,G1,...,G1252] of graphs as named in the Graph Atlas.

#### Classic
- balanced_tree(r, h[, create_using]) Return the perfectly balanced r-tree of height h.
- barbell_graph(m1, m2[, create_using]) Return the Barbell Graph: two complete graphs connected by a path.
- complete_graph(n[, create_using]) Return the complete graph K_n with n nodes.
- complete_multipartite_graph(*block_sizes) Returns the complete multipartite graph with the specified block sizes.
- circular_ladder_graph(n[, create_using]) Return the circular ladder graph CL_n of length n.
- cycle_graph(n[, create_using]) Return the cycle graph C_n over n nodes.
- dorogovtsev_goltsev_mendes_graph(n[, ...]) Return the hierarchically constructed Dorogovtsev-Goltsev-Mendes graph.
- empty_graph([n, create_using]) Return the empty graph with n nodes and zero edges.
- grid_2d_graph(m, n[, periodic, create_using]) Return the 2d grid graph of mxn nodes, each connected to its nearest neighbors.
- grid_graph(dim[, periodic]) Return the n-dimensional grid graph.
- hypercube_graph(n) Return the n-dimensional hypercube.
- ladder_graph(n[, create_using]) Return the Ladder graph of length n.
- lollipop_graph(m, n[, create_using]) Return the Lollipop Graph; Km connected to Pn.
- null_graph([create_using]) Return the Null graph with no nodes or edges.
- path_graph(n[, create_using]) Return the Path graph P_n of n nodes linearly connected by n-1 edges.
- star_graph(n[, create_using]) Return the Star graph with n+1 nodes: one center node, connected to n outer nodes.
- trivial_graph([create_using]) Return the Trivial graph with one node (with integer label 0) and no edges.
- wheel_graph(n[, create_using]) Return the wheel graph: a single hub node connected to each node of the (n-1)-node cycle graph.

#### Expanders
- margulis_gabber_galil_graph(n[, create_using]) Return the Margulis-Gabber-Galil undirected MultiGraph on n^2 nodes.
- chordal_cycle_graph(p[, create_using]) Return the chordal cycle graph on p nodes.

#### Small
- make_small_graph(graph_description[, ...]) Return the small graph described by graph_description.
- LCF_graph(n, shift_list, repeats[, create_using]) Return the cubic graph specified in LCF notation.
- bull_graph([create_using]) Return the Bull graph.
- chvatal_graph([create_using]) Return the Chvátal graph.
- cubical_graph([create_using]) Return the 3-regular Platonic Cubical graph.
- desargues_graph([create_using]) Return the Desargues graph.
- diamond_graph([create_using]) Return the Diamond graph.
- dodecahedral_graph([create_using]) Return the Platonic Dodecahedral graph.
- frucht_graph([create_using]) Return the Frucht Graph.
- heawood_graph([create_using]) Return the Heawood graph, a (3,6) cage.
- house_graph([create_using]) Return the House graph (square with triangle on top).
- house_x_graph([create_using]) Return the House graph with a cross inside the house square.
- icosahedral_graph([create_using]) Return the Platonic Icosahedral graph.
- krackhardt_kite_graph([create_using]) Return the Krackhardt Kite Social Network.
- moebius_kantor_graph([create_using]) Return the Moebius-Kantor graph.
- octahedral_graph([create_using]) Return the Platonic Octahedral graph.
- pappus_graph() Return the Pappus graph.
- petersen_graph([create_using]) Return the Petersen graph.
- sedgewick_maze_graph([create_using]) Return a small maze with a cycle.
- tetrahedral_graph([create_using]) Return the 3-regular Platonic Tetrahedral graph.
- truncated_cube_graph([create_using]) Return the skeleton of the truncated cube.
- truncated_tetrahedron_graph([create_using]) Return the skeleton of the truncated Platonic tetrahedron.
- tutte_graph([create_using]) Return the Tutte graph.

#### Random Graphs
...

#### Degree Sequence
Generate graphs with a given degree sequence or expected degree sequence.
...

#### Random Clustered
Generate graphs with given degree and triangle sequence.
- random_clustered_graph(joint_degree_sequence) Generate a random graph with the given joint independent edge degree and triangle degree sequence.

#### Directed
Generators for some directed graphs, including growing network (GN) graphs and scale-free graphs.
- gn_graph(n[, kernel, create_using, seed]) Return the growing network (GN) digraph with n nodes.
- gnr_graph(n, p[, create_using, seed]) Return the growing network with redirection (GNR) digraph with n nodes and redirection probability p.
- gnc_graph(n[, create_using, seed]) Return the growing network with copying (GNC) digraph with n nodes.
- scale_free_graph(n[, alpha, beta, gamma, . . . ]) Returns a scale-free directed graph.

#### Geometric
- random_geometric_graph(n, radius[, dim, pos]) Returns a random geometric graph in the unit cube.
- geographical_threshold_graph(n, theta[, . . . ]) Returns a geographical threshold graph.
- waxman_graph(n[, alpha, beta, L, domain]) Return a Waxman random graph.
- navigable_small_world_graph(n[, p, q, r, . . . ]) Return a navigable small-world graph.

#### Line Graph
The line graph of a graph G has a node for each edge in G and an edge joining those nodes if the two edges in G share a common node. For directed graphs, nodes are adjacent exactly when the edges they represent form a directed path of length two.
- line_graph(G[, create_using]) Returns the line graph of the graph or digraph G.

#### Ego Graph
- ego_graph(G, n[, radius, center, ...]) Returns induced subgraph of neighbors centered at node n within a given radius.

#### Stochastic
- stochastic_graph(G[, copy, weight]) Returns a right-stochastic representation of the directed graph G.

#### Intersection
- uniform_random_intersection_graph(n, m, p[, ...]) Return a uniform random intersection graph.
- k_random_intersection_graph(n, m, k) Return a intersection graph with randomly chosen attribute sets for each node that are of equal size (k).
- general_random_intersection_graph(n, m, p) Return a random intersection graph with independent probabilities for connections between node and attribute sets.

#### Social Networks
- karate_club_graph() Return Zachary’s Karate Club graph.
- davis_southern_women_graph() Return Davis Southern women social network.
- florentine_families_graph() Return Florentine families graph.

#### Coummunity
Generators for classes of graphs used in studying social networks.
- caveman_graph(l, k) Returns a caveman graph of l cliques of size k.
- connected_caveman_graph(l, k) Returns a connected caveman graph of l cliques of size k.
- relaxed_caveman_graph(l, k, p[, seed]) Return a relaxed caveman graph.
- random_partition_graph(sizes, p_in, p_out[, ...]) Return the random partition graph with a partition of sizes.
- planted_partition_graph(l, k, p_in, p_out[, ...]) Return the planted l-partition graph.
- gaussian_random_partition_graph(n, s, v, ...) Generate a Gaussian random partition graph.

#### Non Isomorphic Trees
- nonisomorphic_trees(order[, create]) Returns a list of nonisomporphic trees
- number_of_nonisomorphic_trees(order) Returns the number of nonisomorphic trees


### Linear algebra(线性代数)
#### Graph matrix
- adjacency_matrix(G[, nodelist, weight]) Return adjacency matrix of G.
- incidence_matrix(G[, nodelist, edgelist, . . . ]) Return incidence matrix of G.

#### Laplacian Matrix
- laplacian_matrix(G[, nodelist, weight]) Return the Laplacian matrix of G.
- normalized_laplacian_matrix(G[, nodelist, ...]) Return the normalized Laplacian matrix of G.
- directed_laplacian_matrix(G[, nodelist, ...]) Return the directed Laplacian matrix of G.

#### Spectrum
Eigenvalue spectrum of graphs.
- laplacian_spectrum(G[, weight]) Return eigenvalues of the Laplacian of G
- adjacency_spectrum(G[, weight]) Return eigenvalues of the adjacency matrix of G.

#### Algebraic Connectivity(代数连通度)
Algebraic connectivity and Fiedler vectors of undirected graphs.
- algebraic_connectivity(G[, weight, ...]) Return the algebraic connectivity of an undirected graph.
- fiedler_vector(G[, weight, normalized, tol, ...]) Return the Fiedler vector of a connected undirected graph.
- spectral_ordering(G[, weight, normalized, ...]) Compute the spectral_ordering of a graph.

#### Attribute Matrices
- attr_matrix(G[, edge_attr, node_attr, ...]) Returns a NumPy matrix using attributes from G.
- attr_sparse_matrix(G[, edge_attr, ...]) Returns a SciPy sparse matrix using attributes from G.

### Converting to and from other data formats
#### ToNetworkX Graph
- to_networkx_graph(data[, create_using, ...]) Make a NetworkX graph from a known data structure.
```
d={0: {1: {'weight':1}}} # dict-of-dicts single edge (0,1)
G=nx.from_dict_of_dicts(d)
```

#### Dictionaries
- to_dict_of_dicts(G[, nodelist, edge_data]) Return adjacency representation of graph as a dictionary of dictionaries.
- from_dict_of_dicts(d[, create_using, ...]) Return a graph from a dictionary of dictionaries.

#### Lists
- to_dict_of_lists(G[, nodelist]) Return adjacency representation of graph as a dictionary of lists.
- from_dict_of_lists(d[, create_using]) Return a graph from a dictionary of lists.
- to_edgelist(G[, nodelist]) Return a list of edges in the graph.
- from_edgelist(edgelist[, create_using]) Return a graph from a list of edges.

#### Numpy
```
import numpy
a = numpy.reshape(numpy.random.random_integers(0,1,size=100),(10,10))
D = nx.DiGraph(a)
# or equivalently
D = nx.to_networkx_graph(a,create_using=nx.DiGraph())
```
- to_numpy_matrix(G[, nodelist, dtype, order, ...]) Return the graph adjacency matrix as a NumPy matrix.
- to_numpy_recarray(G[, nodelist, dtype, order]) Return the graph adjacency matrix as a NumPy recarray.
- from_numpy_matrix(A[, parallel_edges, . ...]) Return a graph from numpy matrix.

#### Scipy
- to_scipy_sparse_matrix(G[, nodelist, dtype, ...]) Return the graph adjacency matrix as a SciPy sparse matrix.
- from_scipy_sparse_matrix(A[, ...]) Creates a new graph from an adjacency matrix given as a SciPy sparse matrix.

#### Pandas
- to_pandas_dataframe(G[, nodelist, ...]) Return the graph adjacency matrix as a Pandas DataFrame.
- from_pandas_dataframe(df, source, target[, ...]) Return a graph from Pandas DataFrame.

### Reading and writing graphs
#### Adjacency List
Adjacency list format is useful for graphs without data associated with nodes or edges and for nodes that can be meaningfully represented as strings.
- read_adjlist(path[, comments, delimiter, ...]) Read graph in adjacency list format from path.
```
G=nx.path_graph(4)
nx.write_adjlist(G, "test.adjlist")
G=nx.read_adjlist("test.adjlist")
"test.adjlist":
0 1
1 2
2 3
3

fh=open("test.adjlist", 'rb')
G=nx.read_adjlist(fh)

nx.write_adjlist(G,"test.adjlist.gz")
G=nx.read_adjlist("test.adjlist.gz")

G=nx.read_adjlist("test.adjlist", nodetype=int)

G=nx.read_adjlist("test.adjlist", create_using=nx.DiGraph())
```
- write_adjlist(G, path[, comments, ...]) Write graph G in single-line adjacency-list format to path.
```
fh=open("test.adjlist",'wb')
nx.write_adjlist(G, fh)
```
- parse_adjlist(lines[, comments, delimiter, ...]) Parse lines of a graph adjacency list representation.
```
lines = ['1 2 5',
		 '2 3 4',
		 '3 5',
		 '4',
		 '5']
G = nx.parse_adjlist(lines, nodetype = int)
G.edges()
[Out]: [(1, 2), (1, 5), (2, 3), (2, 4), (3, 5)]

```
- generate_adjlist(G[, delimiter]) Generate a single line of the graph G in adjacency list format.
```
G = nx.lollipop_graph(4, 3)
for line in nx.generate_adjlist(G):
	print(line)

```

#### Multiline Adjacency List
The first label in a line is the source node label followed by the node degree d. The next d lines are target node labels and optional edge data. That pattern repeats for all nodes in the graph.
- read_multiline_adjlist(path[, comments, ...]) Read graph in multi-line adjacency list format from path.
- write_multiline_adjlist(G, path[, ...]) Write the graph G in multiline adjacency list format to path
- parse_multiline_adjlist(lines[, comments, ...]) Parse lines of a multiline adjacency list representation of a graph.
- generate_multiline_adjlist(G[, delimiter]) Generate a single line of the graph G in multiline adjacency list format.

#### Edge List
With the edgelist format simple edge data can be stored but node or graph data is not. There is no way of representing isolated nodes unless the node has a self-loop edge.
You can read or write three formats of edge lists with these functions:
1) Node pairs with no data:
`1 2`
2) Python dictionary as data:
`1 2 {'weight':7, 'color':'green'}`
3) Arbitrary data:
`1 2 7 green`

- read_edgelist(path[, comments, delimiter, ...]) Read a graph from a list of edges.
```
G=nx.read_edgelist("test.edgelist")

fh=open("test.edgelist", 'rb')
G=nx.read_edgelist(fh)
fh.close()

G=nx.read_edgelist("test.edgelist", nodetype=int)
G=nx.read_edgelist("test.edgelist",create_using=nx.DiGraph())
```
- write_edgelist(G, path[, comments, ...]) Write graph as a list of edges.
```
G=nx.path_graph(4)
nx.write_edgelist(G, "test.edgelist")

fh=open("test.edgelist",'wb')
nx.write_edgelist(G, fh)

nx.write_edgelist(G, "test.edgelist.gz")
nx.write_edgelist(G, "test.edgelist.gz", data=False)

G.add_edge(1,2,weight=7,color='red')
nx.write_edgelist(G,'test.edgelist',data=False)
nx.write_edgelist(G,'test.edgelist',data=['color'])
nx.write_edgelist(G,'test.edgelist',data=['color','weight'])
```
- read_weighted_edgelist(path[, comments, ...]) Read a graph as list of edges with numeric weights.
- write_weighted_edgelist(G, path[, comments, ...]) Write graph G as a list of edges with numeric weights.
- generate_edgelist(G[, delimiter, data]) Generate a single line of the graph G in edge list format.
- parse_edgelist(lines[, comments, delimiter, ...]) Parse lines of an edge list representation of a graph.

#### GEXF
GEXF (Graph Exchange XML Format) is a language for describing complex network structures, their associated data
and dynamics.
- read_gexf(path[, node_type, relabel, version]) Read graph in GEXF format from path.
- write_gexf(G, path[, encoding, prettyprint, ...]) Write G in GEXF format to path.
- relabel_gexf_graph(G) Relabel graph using “label” node keyword for node label.

#### GML
“GML, the Graph Modelling Language, is our proposal for a portable file format for graphs. GML’s key features are portability, simple syntax, extensibility and flexibility. A GML file consists of a hierarchical key-value lists. Graphs can be annotated with arbitrary data structures. The idea for a common file format was born at the GD‘95; this proposal is the outcome of many discussions. GML is the standard file format in the Graphlet graph editor system. It has been overtaken and adapted by several other systems for drawing graphs.”

- read_gml(path[, label, destringizer]) Read graph in GML format from path.
- write_gml(G, path[, stringizer]) Write a graph G in GML format to the file or file handle path.
- parse_gml(lines[, label, destringizer]) Parse GML graph from a string or iterable.
- generate_gml(G[, stringizer]) Generate a single entry of the graph G in GML format.
- literal_destringizer(rep) Convert a Python literal to the value it represents.
- literal_stringizer(value) Convert a value to a Python literal in GML representation.

#### Pickle
Pickles are a serialized byte stream of a Python object. This format will preserve Python objects used as nodes or edges.
- read_gpickle(path) Read graph object in Python pickle format.
- write_gpickle(G, path[, protocol]) Write graph in Python pickle format.

#### GraphML
“GraphML is a comprehensive and easy-to-use file format for graphs. It consists of a language core to describe the structural properties of a graph and a flexible extension mechanism to add application-specific data. Its main features include support of
• directed, undirected, and mixed graphs,
• hypergraphs,
• hierarchical graphs,
• graphical representations,
• references to external data,
• application-specific attribute data, and
• light-weight parsers.
Unlike many other file formats for graphs, GraphML does not use a custom syntax. Instead, it is based on XML and hence ideally suited as a common denominator for all kinds of services generating, archiving, or processing graphs.”

- read_graphml(path[, node_type]) Read graph in GraphML format from path.
- write_graphml(G, path[, encoding, prettyprint]) Write G in GraphML XML format to path

#### JSON(json_graph.)
- node_link_data(G[, attrs]) Return data in node-link format that is suitable for JSON serialization and use in Javascript documents.
- node_link_graph(data[, directed, ...]) Return graph from node-link data format.
- adjacency_data(G[, attrs]) Return data in adjacency format that is suitable for JSON serialization and use in Javascript documents.
- adjacency_graph(data[, directed, ...]) Return graph from adjacency data format.
- tree_data(G, root[, attrs]) Return data in tree format that is suitable for JSON serialization and use in Javascript documents.
- tree_graph(data[, attrs]) Return graph from tree data format.
```
from networkx.readwrite import json_graph
G = nx.Graph([(1,2)])
data = json_graph.node_link_data(G)
# To serialize with json
import json
s = json.dumps(data)
```

#### LEDA
LEDA is a C++ class library for efficient data types and algorithms.
See http://www.algorithmic-solutions.info/leda_guide/graphs/leda_native_graph_fileformat.html

- read_leda(path[, encoding]) Read graph in LEDA format from path.
- parse_leda(lines) Read graph in LEDA format from string or iterable.

#### YAML
[YAML is a data serialization format designed for human readability and interaction with scripting languages.](http://www.yaml.org)
- read_yaml(path) Read graph in YAML format from path.
- write_yaml(G, path[, encoding]) Write graph G in YAML format to path.

#### SparseGraph6
[graph6 and sparse6 are formats for storing undirected graphs in a compact manner, using only printable ASCII characters. Files in these formats have text type and contain one line per graph.](http://cs.anu.edu.au/~bdm/data/formats.txt)

- parse_graph6(string) Read a simple undirected graph in graph6 format from string.
- read_graph6(path) Read simple undirected graphs in graph6 format from path.
- generate_graph6(G[, nodes, header]) Generate graph6 format string from a simple undirected graph.
- write_graph6(G, path[, nodes, header]) Write a simple undirected graph to path in graph6 format.
- parse_sparse6(string) Read an undirected graph in sparse6 format from string.
- read_sparse6(path) Read an undirected graph in sparse6 format from path.
- generate_sparse6(G[, nodes, header]) Generate sparse6 format string from an undirected graph.
- write_sparse6(G, path[, nodes, header]) Write graph G to given path in sparse6 format.

#### Pajek
http://vlado.fmf.uni-lj.si/pub/networks/pajek/doc/draweps.htm
- read_pajek(path[, encoding]) Read graph in Pajek format from path.
- write_pajek(G, path[, encoding]) Write graph in Pajek format to path.
- parse_pajek(lines) Parse Pajek format graph from string or iterable.

#### GIS Shapefile
- read_shp(path[, simplify]) Generates a networkx.DiGraph from shapefiles.
- write_shp(G, outdir) Writes a networkx.DiGraph to two shapefiles, edges and nodes.

### Drawing
#### Matplotlib
- draw(G[, pos, ax, hold]) Draw the graph G with Matplotlib.
- draw_networkx(G[, pos, arrows, with_labels]) Draw the graph G using Matplotlib.
- draw_networkx_nodes(G, pos[, nodelist, ...]) Draw the nodes of the graph G.
- draw_networkx_edges(G, pos[, edgelist, ...]) Draw the edges of the graph G.
- draw_networkx_labels(G, pos[, labels, ...]) Draw node labels on the graph G.
- draw_networkx_edge_labels(G, pos[, ...]) Draw edge labels.
- draw_circular(G, **kwargs) Draw the graph G with a circular layout.
- draw_random(G, **kwargs) Draw the graph G with a random layout.
- draw_spectral(G, **kwargs) Draw the graph G with a spectral layout.
- draw_spring(G, **kwargs) Draw the graph G with a spring layout.
- draw_shell(G, **kwargs) Draw networkx graph with shell layout.
- draw_graphviz(G[, prog]) Draw networkx graph with graphviz layout.

```
G=nx.dodecahedral_graph()
nx.draw(G)
nx.draw(G,pos=nx.spring_layout(G)) # use spring layout

import matplotlib.pyplot as plt
import networkx as nx
G=nx.dodecahedral_graph()
nx.draw(G) # networkx draw()
plt.draw() # pyplot draw()
```
#### Graphviz AGraph (dot)
