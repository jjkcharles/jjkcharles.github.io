---
layout: post
title: Tackling SQL Compatibility
subtitle: Solving an 'Update' Query Mystery in MS SQL & PostgreSQL
author: jjk_charles
categories: programming
tags: sql postgresql
sidebar: []
---

Over the last few months, our team at work has been busy porting an application developed to work on top of MS SQL & Oracle as data stores to work with PostgreSQL as the data store. While porting the code that interfaces with Oracle proved to be relatively simpler, the MS SQL bit proved to be a difficult one - especially with an `Update` query which was behaving in an unexpected manner when ran in PostgreSQL.  
 
### Approach

The task spanned across several APIs and background processes (jobs) that our team had developed over the years. However, the underlying approach for both had been the same.  
#### For Oracle -> PostgreSQL  
We rewrote the queries to be compatible across both Oracle and PostgreSQL. This involved making changes like below (not exhaustive)  
 
1. Not using oracle specific date/time handling (e.g., sysdate, nvl etc.)  
2. Not using oracle specific functions - in some cases replacing it with a different function, or creating a custom functions in PostgreSQL to mimic Oracle functions
3. Replacing JOINs that had Oracle Join operator and replacing them with ANSI join syntax  
 
#### For MS SQL -> PostgreSQL  
For this, the team opted to use the same query across both DBs where it was possible, and simply maintain two versions of other queries as it turned counter-productive for some of the complex queries we had to be made compatible across both MS SQL and PostgreSQL.  

### Problematic Scenario

While we faced numerous challenges, nothing proved as difficult as the one we were dealing with a certain `update` query (this was part of MS SQL).  
 
Below is what the update query does,  
```  
Update the records in a table with data from few other tables, based on match found using the target table's primary key  
```  
 
As simple as it sounds, it didn't work the way we wanted it to. We started noticing that the updates left some incorrect data in the target table. And, we didn't see much of a pattern to it. Primarily because, it is transient data which gets updated by other processes as soon as this problematic update completes.  
 
We started isolating each process to see which one introduces the bad data - and eventually landed on the problematic update.  
 
But, it wasn't until two days from then, a smart dev. in the team identifed a solution. We still weren't entirerly sure what was wrong with that query, we just knew that PostgreSQL is behaving differently when that query executes.  

### Analysis

This got me interested in digging around what is causing this.  
 
OK, I created some simple SQLFiddles to replicate the scenario. The premise here is, we have two tables A & B. And, A is the target whose "groupName" column needs to be updated with value of B tables jobID column prefixed with "g-", if and only if A's id matches B's jobID.  
 
```sql  
create table A  
(  
id int not null,  
name varchar(25),  
groupName varchar(10)  
);  
 
create table B  
(  
id int not null,  
jobId int not null  
);  
```  

##### Sample Data

  The data loaded into those tables:
 
###### Table A

| id | name |
|--|--|
| 1 | Alex |
| 2 | Ben |
| 3 | Cathy |
| 4 | Doug |
| 5 | Elsa |

###### Table B

| id | jobId |
|--|--|
| 1 | 1 |
| 2 | 1 |
| 3 | 2 |
| 4 | 3 |
| 5 | 2 |
 
Below is the problematic query:  
 
```sql  
UPDATE A  
SET groupName = 'g-'||b1.jobId  
FROM A a1  
INNER JOIN B b1 on [a1.id](http://a1.id) = [b1.id](http://b1.id)  
AND [a1.id](http://a1.id) = b1.jobId;  
```  

### Revelation

After my futile attempt at trying few things without any luck, I quickly realized I am running out of options and the last resort I have is to refer to the docs! ([relevant XKCD](https://xkcd.com/293/))  
 
![XKCD about reading the manual](https://imgs.xkcd.com/comics/rtfm.png "Read the manual")  

After going through the docs, it became clear that the query is semantically valid across both DB Engines but they interpret the query differently.

Below are the sections of most interest to us,

From [MS SQL Server Documentation](https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql?view=sql-server-ver16):
```sql
-- Syntax for SQL Server and Azure SQL Database  

[ WITH <common_table_expression> [...n] ]  
UPDATE  
    [ TOP ( expression ) [ PERCENT ] ]  
    { { table_alias | <object> | rowset_function_limited  
         [ WITH ( <Table_Hint_Limited> [ ...n ] ) ]  
      }  
      | @table_variable      
    }  
    SET  
        { column_name = { expression | DEFAULT | NULL }  
          | { udt_column_name.{ { property_name = expression  
                                | field_name = expression }  
                                | method_name ( argument [ ,...n ] )  
                              }  
          }  
          | column_name { .WRITE ( expression , @Offset , @Length ) }  
          | @variable = expression  
          | @variable = column = expression  
          | column_name { += | -= | *= | /= | %= | &= | ^= | |= } expression  
          | @variable { += | -= | *= | /= | %= | &= | ^= | |= } expression  
          | @variable = column { += | -= | *= | /= | %= | &= | ^= | |= } expression  
        } [ ,...n ]  
 
    [ <OUTPUT Clause> ]  
    [ FROM{ <table_source> } [ ,...n ] ]  
    [ WHERE { <search_condition>  
            | { [ CURRENT OF  
                  { { [ GLOBAL ] cursor_name }  
                      | cursor_variable_name  
                  }  
                ]  
              }  
            }  
    ]  
    [ OPTION ( <query_hint> [ ,...n ] ) ]  
[ ; ]  
 
<object> ::=  
{  
    [ server_name . database_name . schema_name .  
    | database_name .[ schema_name ] .  
    | schema_name .  
    ]  
    table_or_view_name}
```

*Snip of syntax definition for relevant "From" clause used in the query*

>**FROM \<table_source\>**
>
>Specifies that a table, view, or derived table source is used to provide the criteria for the update operation.

___

From [PostgreSQL documentation](https://www.postgresql.org/docs/current/sql-update.html):
```sql
[ WITH [ RECURSIVE ] _`with_query`_ [, ...] ]
UPDATE [ ONLY ] _`table_name`_ [ * ] [ [ AS ] _`alias`_ ]
    SET { _`column_name`_ = { _`expression`_ | DEFAULT } |
          ( _`column_name`_ [, ...] ) = [ ROW ] ( { _`expression`_ | DEFAULT } [, ...] ) |
          ( _`column_name`_ [, ...] ) = ( _`sub-SELECT`_ )
        } [, ...]
    [ FROM _`from_item`_ [, ...] ]
    [ WHERE _`condition`_ | WHERE CURRENT OF _`cursor_name`_ ]
    [ RETURNING * | _`output_expression`_ [ [ AS ] _`output_name`_ ] [, ...] ]
```

*Snip of syntax definition for relevant "From" clause used in the query*
>
>**from_item**
>
>A table expression allowing columns from other tables to appear in the WHERE condition and update expressions. This uses the same syntax as the FROM clause of a SELECT statement; for example, an alias for the table name can be specified. Do not repeat the target table as a from_item unless you intend a self-join (in which case it must appear with an alias in the from_item).

---

As could be seen, MS SQL uses the FROM clause to specify tables that can be used in the Update query's where clause. Also, a single occurance of the target table in the FROM list implicitly refers to the table being updated.

Whereas in PostgreSQL, the FROM clause bring in "additional" tables that can be brought into the WHERE clause. Infact, docs explicitly calls out the below,

> Do not repeat the target table as a from_item unless you intend a self-join

### Solution

So, basically within our query since we repeat the `target` table in FROM clause, PostgreSQL is doing a self-join with no join condition causing the entire table to be updated.

The fix turned out to be simple, it can be one of the two depending on what you wanted to achieve,

#### Option 1
Introduce an alias for the target column, and ensure that the self-join matches only those records that have the same id.

```sql
UPDATE A a0
SET groupName = 'g-'||b1.jobId
FROM A a1
INNER JOIN B b1 on a1.id = b1.id
  AND a1.id = b1.jobId
WHERE a0.id = a1.id;
```

#### Option 2
Get rid of target column from the FROM clause, and ensure that the self-join matches only those records that have the same id/jobid condition we are interested in.

```sql
UPDATE A a1
SET groupName = 'g-'||b1.jobId
FROM B b1
WHERE a1.id = b1.id
and a1.id = b1.jobId;
```

In our case, since the self-join is not necessary, it only adds overhead to the query. So, we went with Option 2.

>**Note:** These queries in the solution are meant to be used in PostgreSQL and are not functionally compatible when used in MS SQL.