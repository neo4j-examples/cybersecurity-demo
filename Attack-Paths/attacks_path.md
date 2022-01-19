# Building an attack path graph from a network graph

Given a graph of network connections, we create a named graph so that we can work on it :
```cql
CALL gds.graph.create("attackPathGraph", '*', 
  { relType: {
    type: '*',
    orientation: 'UNDIRECTED'
  }});
```

Build a graph of all shortest attack paths with [Dijkstra algorithm](https://neo4j.com/docs/graph-data-science/current/algorithms/dijkstra-source-target/) to a specific crown jewel:
And then we create `POSSIBLE_ATTACK` relationships:

```cql
MATCH (external:ExternalFacingNode)
MATCH (target:Resource) WHERE target.name='CustomerDatabase' 
CALL gds.shortestPath.dijkstra.stream("attackPathGraph", 
{ sourceNode: id(external),
  targetNode: target, 
  path: true})
YIELD sourceNode,targetNode,nodeIds 
WITH nodeIds
UNWIND apoc.coll.pairsMin(gds.util.asNodes(nodeIds)) AS pair
WITH pair[0] AS toNode, pair[1] AS fromNode
MERGE (toNode) <- [:POSSIBLE_ATTACK] - (fromNode)
```

Create a `centralityGraph` to calculate the score of the centrality using [Betweenness algorithm](https://neo4j.com/docs/graph-data-science/current/algorithms/betweenness-centrality/) for all `POSSIBLE_ATTACK` relationships:
```cql
CALL gds.graph.create('centralityGraph', '*',
{ relType: {
  type: '*',
  orientation: 'UNDIRECTED'
}});

CALL gds.betweenness.stream('centralityGraph', {})
YIELD nodeId, score
WITH gds.util.asNode(nodeId) as n, score AS betweenness
RETURN n.name, betweenness ORDER BY betweenness DESC
```
Finally, drop the graph projections:
```
CALL gds.graph.drop("attackPathGraph");
CALL gds.graph.drop("centralityGraph");
```