# Irish Constituencies Neo4j Database
###### Oliver Arnold, G00311681

## Introduction
With this project I wanted to populate a Neo4j database with enough statistics, so that I could query it to find out lots of interesting facts about the build-up to and the aftermath of the Irish General Election in 2016.

## Database
### Csv Files
When creating my database, I used 3 CSV files to populate my database with nodes. Two csv files are variations of the candidates.csv file found here: "https://github.com/storyful/irish-elections". As this csv had lots of information I didn't need, I had to delete many columns. As well as that, some fields had null values, and so I had to source that information myself. The first file I used was the ElectedTds.csv file, this file is made up of only the required information for Candidates that got elected. The second file I made was FullCandidatesList. This contains the required information on all candidates running for office in 2016. Including a candidate that was missing from the original source file. 
The PartyPolicies.csv file was created entirely by myself. It contains the names of all running parties, their political stance, and how they stand on free college tuition for students. The political stance I sourced mainly from each party's wikipedia page, and in cases where a party were deemed as center-right, I made an educated decision on either right or center as their main stance. While the information about college tuition fees was tediously gathered from smartvote.ie. 

For the actual queries I used in creating my DB, please see the QuerySetupDB.txt file. 

###Constituencies
Each constituency was created by using the constituency label in the TD nodes. I search through the TDs, and where a constituency that hasn't been used yet is found, a new node is created. 
For the actual queries I used in creating my DB, please see the QuerySetupDB.txt file. 

###Relationships
To link nodes through relationships, I used the party and constituency labels in the TD node, and created a relationship where the label matched the name of the inteded node. 
For the actual queries I used in creating my DB, please see the QuerySetupDB.txt file. 

## Queries
My three interesting queries, i've made pluck very varied information from the DB. One finds how the top three parties feel about third level college fees. My second query finds out which constituency is the least competitive constituency in the country. My final query finds out which way Ireland sways on the political spectrum. 

#### Third level education fees
This query discovers how the top three elected parties feel about government funded college tuition fees. 
I found this query pretty interesting as none of the three parties agree with one another on the issue. 
```cypher
MATCH (m:Party)-[r:Has_Member]->(n:TD) 
where n.elected="true"
WITH m, (COUNT(m)) as noSeats, collect(distinct m) as Parties
order by noSeats desc
limit 3
unwind Parties as party
return party.CollegeTuitionPolicy
```

#### Least competitive constituency, and it's Constituents
This query finds out which constituency has the least amount of candidates running for office. This query can be modified to show the most competitive by changing asc, to desc on the third line
```cypher
MATCH (m:Constituency)-[r:Had_Running]->(x:TD) 
with m, count(distinct x) as runnerCount, collect(distinct m) as places
order by runnerCount asc limit 1
unwind places as constituency
match (m:Constituency)-[r:Had_Running]->(x:TD) 	
where m=constituency
return r
```

#### How many TDS fall into each political category
This query counts how many TDs fall into each political category (Right wing, Left wing, Central).
This query was a little more complex as the Political stance is stored in the Party node, not the TD node
```cypher
match(m:TD)-[r]->(n:Party)
where m.elected="true"
with n, count(distinct m) as tdCount, collect(m) as tds
order by tdCount desc
unwind tds as td
match(td)-[r]->(n:Party)
with SUM(CASE WHEN n.Stance="Right" THEN 1 ELSE 0 END) AS Right_Wing,
SUM(CASE WHEN n.Stance="Left" THEN 1 ELSE 0 END) AS Left_Wing,
SUM(CASE WHEN n.Stance="Centre" THEN 1 ELSE 0 END) AS Central
return Right_Wing, Left_Wing, Central
```

## References
1. [Neo4J website](http://neo4j.com/), the website of the Neo4j database.
2. [Irish Politics Github](https://github.com/storyful/irish-elections), the source of my initial CSV file
3. [Smart Vote](http://www.smartvote.ie/), my source on education fees
