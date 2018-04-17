# Requirements
It requires networkx python library to execute.
So first install networkx library.

`pip install networkx`

# Usage

`python [file.py] [input_file_path] [output_file_path]`
- input_file_path : path of the graph input file.
- output_file_path : path of the file where output of the program will be stored.

# Algorithm
- Consider a directed network G(V, E) over a set of nodes V and a set of edges E. We first convert G(V, E) to an equivalent undirected bipartite graph B(V_in, V_out, E).
- The bipartite graph is built by splitting the node set V into two node sets Vin(-ve) and Vout(+ve), where a node u in G is converted to two nodes u_in and u_out in B, and nodes u_in and u_out are connected to the in-edges and out-edges of node u respectively.
- A matching is a set of edges that share no common node. A matching with the maximum number of edges is called a maximum matching.
  We can find number of driver nodes, with only one maximum bipartite matching, So we don't have enumerate all the maximum matchings.
  
- For a directed network G(V, E), find its corresponding bipartite graph B(V_out, V_in, E). Let the initial matching M = {};
- Find all the alternating paths of all unmatched nodes in V_in, and store the nodes of alternate paths in V_in as `candidate nodes`.                  
`Candidate nodes are nodes, which might become driver nodes`                       
 We used Breadth First Search to find if there exist an augmenting path or not.                                      
  
 ``` 
   def breadthfirstSearch(self, f):
        isAvailable = False
        del self.unmatched[:]
        for node in self.Graph:
            self.d[node] = 0
            if self.marked[node] == None and node < 0:
                self.unmatched.append(node)
        for node in self.unmatched:
            for src in self.Graph.neighbors(node):
                if self.d[src] == 0:
                    self.d[src] = self.d[node] + 1
                    if self.marked[src] == None:
                        isAvailable = True
                    else:
                        self.d[self.marked[src]] = self.d[src] + 1
                        self.unmatched.append(self.marked[src])
        return isAvailable       
 ```
    

- If set of alternate paths contain augmenting paths, expand all augmenting paths and obtain a new matching M’. Clear all candidate nodes, M = M’ and return to step 2.                               
  We used DepthFirstSearch to find new matching M'.                              
  
  
 ``` 
    def depthFirstSearch(self, v):
        for node in self.Graph.neighbors(v):
            if self.d[node] == self.d[v] + 1:
                self.d[node] = 0
                if self.marked[node] == None or self.depthFirstSearch(self.marked[node]):
                    self.marked[node] = v
                    self.marked[v] = node
                    return True
        return False                      
 ```  
        
- If there exists no augmenting path, then the the unmatched nodes are the driver nodes of G.                   
- Now we have all AU, AS and SSSU nodes.                      


  `AU : always unsaturated nodes. The nodes which are unmatched in all maximum matchings.`                                                        
  `AS : always saturated nodes. The nodes which are matched in all maximum matchings.`                                                                             
  `SSSU : sometimes saturated, sometimes unsaturated nodes. The nodes which are saturated in some maximum matchings and unsaturated     in some matchings.` 

- The time complexity of the above algorithm is the same as that of the Hopcroft-Karp algorithm, which is O(N^1/2L).             
```
N_D : Number of driver nodes

Note that in Type-I classification, we classify a node to be
1) Nc_1: critical if in its absence we have to control more driver nodes. For example, remove one node in the middle of a directed path will cause N_D increase. 
2) Nr_1: redundant if in its absence we need to control less driver nodes. For example, remove one leaf node in a star will decrease N_D by 1. 
3) No_1 :ordinary if in its absence we need to control the same number of driver nodes. For example, remove the central hub in a star will not change N_D at all.

In Type-II classification, we classify a node to be 
1) Nc_2: critical if it belongs to all the driver node sets, i.e. we have to always control it. We can rigorously prove that a node is critical if and only if it has zero in-degree, i.e. kin=0. 
2) Nr_2: redundant if it belongs to no driver node sets, so we never need to control them. 
3) No_2: ordinary if it belongs to some but not all driver node sets.
```


# Output 
**The output file will contain a table:**                           
```
Column1 : node                                  
Column2 : type of in_node(-node)                                  
Column3 : Change in total driver nodes if we remove in_node(-node)                                   
Column4 : Change in driver nodes in in_part if we remove in_node(-node)                            
Column5 : type of out_node(+node)                                
Column6 : Change in total driver nodes if we remove out_node(+node)                              
Column7 : Change in driver nodes in in_part if we remove out_node(+node)                              
Column8 : Change in total driver nodes if we remove this node(-node and +node)                            
Column9 : Change in driver nodes in in_part if we remove this node(-node and +node) 
```
**Notation used in above table:**                   
```
Vin(-ve) - copy of the V which is connected to in-edge                
Vin(+ve) - copy of the V which is connected to out-edge          
node - column titled node describes whether it is AU,AS or SSSU            
d_global - change in total number of driver nodes after removing a node          
d_in - change in total number of driver nodes in in-part after removing a node$             
```

**In the console, we will have following output:**
```
Nd : Number of driver nodes
nc_1 : Number of Type-I critical nodes
nr_1 : Number of Type-I redundant nodes
no_1 : Number of Type-I ordinary nodes
nc_2 : Number of Type-II critical nodes
nr_2 : Number of Type-II redundant nodes
no_2 : Number of Type-II ordinary nodes
```
