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


# Output
The output file will contain a table:                                    
Column1: node                                  
Column2: type of in_node(-node)                                  
Column3: Change in total driver nodes if we remove in_node(-node)                                   
Column4: Change in driver nodes in in_part if we remove in_node(-node)                            
Column5: type of out_node(+node)                                
Column6: Change in total driver nodes if we remove out_node(+node)                              
Column7: Change in driver nodes in in_part if we remove out_node(+node)                              
Column8: Change in total driver nodes if we remove this node(-node and +node)                            
Column9: Chnage in driver nodes in in_part if we remove this node(-node and +node)                        
