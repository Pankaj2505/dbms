---
layout: post
title:  "SQL Query Optimization"
date:   2021-03-01 06:08:52 +0530
categories: dbms
permalink: /:categories/:title
---

queryOptimization in SQL server

#### How to measure query statistics
	- SET statistics time on
		- it give how much time query has taken to execute 
	- SET statistics IO on
		- its will provide read information , how many row returned , updated.

#### NonSargability (Search argument ability )
	- What 
		- seeking ability of optimizer to go into index search .
		- There are two ways an optmizer uses index for search 
			- Seek ( like hashing)
			- Scan( time somplexity - big o(n))
			
		- Indexes has B Tree structure (Root,intermediate,Leaf)
		- Scanning mean go to leaf level and scan all the leaf level node)
		- Scanning is very expensive
		
		- Seeking mean look for node from root to intermediate to leaf
		
	- How 
		- when we use any expression in where close like "where month(DOb) like 'March'
		- this expression will update seeking ability to scan.
		- using preceding wild card can change seeking to sacanning like "where lastname like '%hatt'
		- suffix wild card  will not not cause sargebility "Where lastname like 'bh%'"
		
	- How to check if index is sacanning or seeking
		- open execution plan, for each query execution , it will return index seek or scan , hovering mouse cursor over message, will return additional info predicate and object.
		
	- Fix
		- use equivalant expression in where close like "where modifiedDate like 'March'
		- "where lastname like 'Bhatt'
		
	
#### Multi Column Index (Seek not happen in index)
	- What	
		
		- Mulicolumn indexes rule is indexing will start from left to right .
		- so if you are not seeking on column 1 and directly seeking on column 2 then seek will not happen
		
	- Problem 
		- select * from table where lname like 'bhatt'
		- index = nonclusteredindex_fname_mname_lname
		- as we are using lname in where clause , seeking can not happen if left column are not used .
	- Fix
		- 1: create a new index on lname 
		- order of column in index matter , additional point : in select query order of column don't matter
		- 2: update the order of column in index use lastname first if used more often.
		- 3: use all column in where clause in proper order

#### Implicit Conversion
	- What
		- when we are comparing two non simlar data type expression , this impolicit type conversion happen 
	- Probelm 
		- where invoiceNo = n'In232323'
		- if invoice no is varchar type we are converting character to varchar
		- This can change index seeking to scanning
	- Fix	
		- use similar data type
		- use explicit convertor
		
#### Dynamic SQL (Plan Cache floating)(Explicitly parameterzed query)

	- Problem
		- declare sqlstring nvarchar; declare mgrid = 10;
		- set @sqlstring= "select * from table where managerid = " + mgrid;
		- EXECUTE sp_executesql @sqlstring
		
		- here we we run a loop for mgrid , that many execution plan will get generated , so to avoid it use explicite parameterized query 
		
		
	fix
	
		- declare sqlstring nvarchar; declare mgrid = 10;
		- set @sqlstring= "select * from table where managerid = @mgrid ";
		- set @parmeter = '@mgrid int'
		- EXECUTE sp_executesql @sqlstring @parmeter @mgrid=10;

#### query order by clause and index orderby clause
	- let say index has asc order 
	- query is in desc order.
	- problem - query execution will take time
	
#### use join instead of subquery (think when its a large table)
	- problem 
		- select * from table where id in (1,2,3,4,5)
	- fix 
		- create another cte or table and then join them 
	
#### Bookmark lookup 
	- specify column name rather than * in select query, this avoid additional lookup.
	
#### Parameter Sniffing
	- Note: Procedure resuses the plan cache, it means once a procedure is compiled , its execution plan will get saved and it will reuse it in future run.
	- when execute procedure first time for very higly selective query , that select data for shorter period, this will create a Seek plan.
	- next time if we execute procedure for a very big time period , optimizer will enforce the query to use existing seek plan only insted scan plan.
	- what
		- some time our procedure execute faster , some time it execute slower .this is called parameter sniffing 
	- Fix 
		- select * from table where condition option(recompile) 
		- this will tell optimizer to recompile query every time it runs.

	