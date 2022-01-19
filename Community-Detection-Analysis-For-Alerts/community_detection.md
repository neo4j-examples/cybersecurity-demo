# Community Detection for Alerts #

Given a graph of `Alert` entities, we detect the communities using [Louvain](https://neo4j.com/docs/graph-data-science/current/algorithms/louvain):
```cql 
CALL gds.graph.create('communitiesGraph', 'Alert',
{
    relType: {
        type: '*',
        orientation: 'UNDIRECTED'
    }}
);

CALL gds.louvain.write('communitiesGraph',
    { writeProperty: 'communityId'}
);
``` 
Create an `AlertGroup` object representing each community and link the alerts to their community using the `communityId` field that the algorithm populated:
```cql
MATCH (a:Alert)
MERGE (a) - [:IN_GROUP] -> (ag:AlertGroup {name: a.communityId});
``` 
Now we can use the information on the alerts to decide how much we trust the group as a whole, and write that information to the group itself:
```cql 
MATCH (group:AlertGroup) <- [:IN_GROUP] - (e:Alert {from: 'nicecorp'})
SET group.trustworthy='high';
MATCH (group:AlertGroup) <- [:IN_GROUP] - (e:Alert {from: 'evilcorp'})
SET group.trustworthy='low';
```
To view this information:
```cql 
MATCH (a:Alert) - [] -> (ag:AlertGroup ) WHERE ag.trustworthy IS NOT NULL RETURN a, ag;
```
Finally, clean up:
```cql  
CALL gds.graph.drop('communitiesGraph');
```