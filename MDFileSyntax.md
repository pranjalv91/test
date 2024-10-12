<!--- 
Table of content [[_TOC_]] do not remove.
--->
[[_TOC_]]

# Solution phase
---

<!--- 
Mark with [x] symbol current status of the solution.
--->

- [x] Design
- [x] Development
- [ ] Deployment


![obraz.png](/.attachments/obraz-f56d212e-3553-4353-850e-b8913e9e32ba.png)

# Design
---
Below section describes user requirements and features that have been composed out of it.
## User story
---

<!--- 
Enter here a description of user requirement.
--->

> As a user I want to have a report that shows sales orders that have been created in last 7 days. I need to see full description of document type for easier filtering. I also want to know in which country is the selling plant located (ISO code + Full description). Both country and document type should be displayed in english.
> Columns that I want to see are: Client, Sales Document, Sales Document Type, SDT Description, Creation date, Creation time, Sales Document Item, Material Number, Plant, Name, Country Key, Country Name. Data should be gathered daily and stored in dedicated table.


## Features
---

<!--- 
Transform user story into separate features.
--->

1. As a user I want to have a report that shows sales orders that have been created in **last 7 days.** 
2. I need to see _full description_ of document type for easier filtering. 
3. I also want to know in which country is the selling plant located ~~(ISO code + Full description).~~
4. Both country and document type should be displayed in english.
5. Columns that I want to see are:
   *    Client,
   *    Sales Document, 
   *    Sales Document Type, 
   *    SDT Description, 
   *    Creation date, 
   *    Creation time, 
   *    Sales Document Item, 
   *    Material Number, 
   *    Plant, 
   *    Name, 
   *    Country Key, 
   *    Country Name. 
6. Data should be gathered daily and stored in dedicated table.



# Implementation
---
Below section describes all the key technical aspects of implemented solution.

<!--- 
Describe what technical steps have been done to implement the solution.
1. Tables
2. Views
3. Coding
4. Jobs
--->

## Tables

In project we have implemented below table:

| Column              | Data Type | Length |
| ------------------- | --------- | ------ |
| Client              | nvarchar  | 3      |
| Sales Document      | nvarchar  | 10     |
| Sales Document Type | nvarchar  | 4      |
| SDT Description     | nvarchar  | 20     |
| Creation date       | date      |
| Creation time       | time      |
| Sales Document Item | nvarchar  | 6      |
| Material Number     | nvarchar  | 18     |
| Plant               | nvarchar  | 4      |
| Name                | nvarchar  | 30     |
| Country Key         | nvarchar  | 3      |
| Country Name        | nvarchar  | 15     |

Source code for creating above table:

```sql
USE [TEST]
GO

SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[WIKI_TEST](
	[Client] [nvarchar](3) NULL,
	[Sales Document] [nvarchar](10) NULL,
	[Sales Document Type] [nvarchar](4) NULL,
	[SDT Description] [nvarchar](20) NULL,
	[Creation date] [date] NULL,
	[Creation time] [time](7) NULL,
	[Sales Document Item] [nvarchar](6) NULL,
	[Material Number] [nvarchar](18) NULL,
	[Plant] [nvarchar](4) NULL,
	[Name] [nvarchar](30) NULL,
	[Country Key] [nvarchar](3) NULL,
	[Country Name] [nvarchar](15) NULL
) ON [PRIMARY]
GO
```

## Procedure - data extact

Content of the procedure used for data extracting:

```sql
USE  [LIB_F6A_RTP]

IF OBJECT_ID('tempdb..#WIKI_TEST') IS NOT NULL DROP TABLE #WIKI_TEST;

SELECT
	VBAK.[MANDT: (PK) Client]
      , VBAK.[VBELN: (PK) Sales Document]
	  , VBAK.[AUART: Sales Document Type]
	  , DOCS.[BEZEI: Description]
      , VBAK.[ERDAT_SIMP_DT: (GC) Date on Which Record Was Created]
      , VBAK.[ERZET_SIMP_TM: (GC) Entry time]
      , VBAP.[POSNR: (PK) Sales Document Item]
      , VBAP.[MATNR: Material Number]
	  , VBAP.[WERKS: Plant (Own or External)]
      , PLANT.[NAME1: Name]
      , PLANT.[LAND1: Country Key]
      , COUNTRY.[LANDX: Country Name]
INTO #WIKI_TEST
FROM [bv].[VBAK: Sales Document: Header Data] VBAK
	INNER JOIN [bv].[VBAP: Sales Document: Item Data] VBAP
		ON VBAP.[VBELN: (PK) Sales Document] = VBAK.[VBELN: (PK) Sales Document]
		AND VBAP.[MANDT: (PK) Client] = '430'
	LEFT JOIN [bv].[T001W: Plants/Branches] PLANT
		ON VBAP.[WERKS: Plant (Own or External)] = PLANT.[WERKS: (PK) Plant]
		AND PLANT.[MANDT: (PK) Client] = '430'
	LEFT JOIN [bv].[T005T: Country Names] COUNTRY
		ON COUNTRY.[LAND1: (PK) Country Key] = PLANT.[LAND1: Country Key]
		AND COUNTRY.[MANDT: (PK) Client] = '430'
		AND COUNTRY.[SPRAS: (PK) Language Key] = 'E'
	LEFT JOIN [bv].[TVAKT: Sales Document Types: Texts] DOCS
		ON DOCS.[AUART: (PK) Sales Document Type] = VBAK.[AUART: Sales Document Type]
		AND DOCS.[MANDT: (PK) Client] = '430'
		AND DOCS.[SPRAS: (PK) Language Key] = 'E'
WHERE 
	VBAK.[MANDT: (PK) Client] = '430'
	AND VBAK.[ERDAT_SIMP_DT: (GC) Date on Which Record Was Created] between DateAdd(DD,-7,GETDATE() ) and GETDATE()

SELECT *

FROM #WIKI_TEST
```

My favourite sql statement is `SELECT * FROM`. I use it very often.

## Execution plan
---
<!--- 
Paste a screen of query execution plan.
--->

## Statistics
---

<!--- 
Paste a screen of query execution plan.
--->

# References
---

<!--- 
Link here any User stories, Epics, Tasks etc.
--->

#7358
#7359

# Testing
---

<!--- 
Paste a screen with test results.
--->

# Attachments
---

<!--- 
Section used for storing attachments.
--->
