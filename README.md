##### Summary:

These two little go command line tool allows you to import your emails from either a mail server via IMAP or an mbox file into a neo4j graph database. I have used the model from the email example project that is explained in Chapter 3 of the book [Graph Databases](http://graphdatabases.com/) by Ian Robinson, Jim Webber and Emil Eifrem (which is a truly great introduction to the graph btw)

#### Usage:
```shell
imap2neo4j imapServer imapUsername imapPassword imapMailbox neo4jServer [neo4jUsername] [neo4jPassword] [paging, eg import by batches of 1000] [specific range of messages, eg 5000:*]

E.g.:
imap2neo4j imap.googlemail.com user@gmail.com password Inbox localhost:7474
imap2neo4j imap.googlemail.com user@gmail.com password Inbox localhost:7473 neo4j password
imap2neo4j imap.googlemail.com user@gmail.com password Inbox localhost:7474 - - 10 0:100
imap2neo4j imap.googlemail.com user@gmail.com password Inbox localhost:7473 neo4j password * 500:*

```

```shell
mbox2neo4j /path/to/mbox/file neo4jServer [neo4jUsername] [neo4jPassword]

E.g.:
mbox2neo4j /path/to/mbox/file localhost:7474
mbox2neo4j /path/to/mbox/file localhost:7473 neo4j password
```

When you specify a neo4j username and password a connection will be made over https

It assumes that you have neo4j 2.2+ installed.

It *shouldn't* do any harm to your emails as all it does is read it's headers but you may test it first on a folder with less important mails, just to be sure.

On a Mac you can export your mails to an mbox file with Mac Mail as well as Outlook for Mac. On a Windows you may have to install a "whatever format"-to-mbox converter.

##### Some useful neo4j/Cypher queries:

###### Visualizing the model
```sql
// Create meta graph  
  MATCH (a)-[r]->(b)   
  WITH labels(a) AS a_labels,type(r) AS rel_type,labels(b) AS b_labels   
  UNWIND a_labels as l   
  UNWIND b_labels as l2   
  MERGE (a:Meta_Node {name:l})   
```
and
```sql
// Show meta graph 
MATCH (n:Meta_Node) RETURN n
```
![alt text](https://github.com/tolomaus/email2neo4j/blob/master/images/model.png "model")

###### The longest email threads
```sql
//Biggest email threads
MATCH p=(e:Email)<-[:REPLY*]-(r:Email)<-[]-(sender:Account)
WHERE NOT (e)-[:REPLY]->()
RETURN sender.name, e.subject, Id(e), length(p) - 1 AS depth
ORDER BY depth DESC
LIMIT 100
```
This will return a table with the id's of the starting emails which can be used in the queries below

```sql
// Email thread by email id
MATCH p=(n:Email)<-[:REPLY*]-(:Email)
WHERE id(n)=123
RETURN p
```
![alt text](https://github.com/tolomaus/email2neo4j/blob/master/images/emailthread.png "Email thread")

```sql
// Email thread by email id now with all accounts
MATCH p=(:Account)-[]-(n:Email)<-[:REPLY*]-(:Email)-[]-(:Account)
WHERE id(n)=135256
RETURN p
```
![alt text](https://github.com/tolomaus/email2neo4j/blob/master/images/emailthreadwithaccounts.png "Email thread with accounts")

