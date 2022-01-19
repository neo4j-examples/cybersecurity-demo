# Deep Graph Analysis 

The following Cypher query returns all systems potentially impacted by a phishing attack. We check the `phisphingEmail` message sent to the `User` and its connections after the attacked user is impersonated.  We find all messages sent to connections within the maximum 6 hops which will cover the most of the network, as suggested by [six degrees of separation](https://en.wikipedia.org/wiki/Six_degrees_of_separation). 

```cql
MATCH  (phishingEmail) - [:WAS_SENT_TO] -> (user:User) - [message:SENT_EMAIL_TO|SENT_SLACK_MESSAGE_TO|SEND_IM_TO*0..6] -  (recipient) 
- [connection:CONNECTED_TO_SYSTEM] -> (potentiallyCompromisedSystem: System)
WHERE connection.date > phishingEmail.sentDate
RETURN potentiallyCompromisedSystem
```