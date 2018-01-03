JIRA 数据库表介绍


Issue Fields
jiraissue表格，这个表格存储了大部分的issue信息。
```
mysql> desc jiraissue;
+----------------------+---------------+------+-----+---------+-------+
| Field                | Type          | Null | Key | Default | Extra |
+----------------------+---------------+------+-----+---------+-------+
| ID                   | decimal(18,0) | NO   | PRI | NULL    |       |
| pkey                 | varchar(255)  | YES  | UNI | NULL    |       |
| PROJECT              | decimal(18,0) | YES  | MUL | NULL    |       |
| REPORTER             | varchar(255)  | YES  |     | NULL    |       |
| ASSIGNEE             | varchar(255)  | YES  | MUL | NULL    |       |
| issuetype            | varchar(255)  | YES  |     | NULL    |       |
| SUMMARY              | varchar(255)  | YES  |     | NULL    |       |
| DESCRIPTION          | longtext      | YES  |     | NULL    |       |
| ENVIRONMENT          | longtext      | YES  |     | NULL    |       |
| PRIORITY             | varchar(255)  | YES  |     | NULL    |       |
| RESOLUTION           | varchar(255)  | YES  |     | NULL    |       |
| issuestatus          | varchar(255)  | YES  |     | NULL    |       |
| CREATED              | datetime      | YES  |     | NULL    |       |
| UPDATED              | datetime      | YES  |     | NULL    |       |
| DUEDATE              | datetime      | YES  |     | NULL    |       |
| RESOLUTIONDATE       | datetime      | YES  |     | NULL    |       |
| VOTES                | decimal(18,0) | YES  |     | NULL    |       |
| WATCHES              | decimal(18,0) | YES  |     | NULL    |       |
| TIMEORIGINALESTIMATE | decimal(18,0) | YES  |     | NULL    |       |
| TIMEESTIMATE         | decimal(18,0) | YES  |     | NULL    |       |
| TIMESPENT            | decimal(18,0) | YES  |     | NULL    |       |
| WORKFLOW_ID          | decimal(18,0) | YES  | MUL | NULL    |       |
| SECURITY             | decimal(18,0) | YES  |     | NULL    |       |
| FIXFOR               | decimal(18,0) | YES  |     | NULL    |       |
| COMPONENT            | decimal(18,0) | YES  |     | NULL    |       |
+----------------------+---------------+------+-----+---------+-------+
```

```
mysql> select id, pkey, project, reporter, assignee, issuetype, summary from jiraissue where pkey='JRA-3166';
+-------+----------+---------+-----------+----------+-----------+---------------------------------+
| id    | pkey     | project | reporter  | assignee | issuetype | summary                         |
+-------+----------+---------+-----------+----------+-----------+---------------------------------+
| 16550 | JRA-3166 |   10240 | mvleeuwen | NULL     | 2         | Database consistency check tool |
+-------+----------+---------+-----------+----------+-----------+---------------------------------+
```

User details
用户相关的一个表格

```
mysql> select user_name, directory_id, display_name, email_address from cwd_user where user_name = 'bright.ma';
+-----------+--------------+--------------+--------------------------+
| user_name | directory_id | display_name | email_address            |
+-----------+--------------+--------------+--------------------------+
| bright.ma |        10700 | Minghui Ma   | bright.ma@example.com |
+-----------+--------------+--------------+--------------------------+

```


Components and versions

```
mysql> desc nodeassociation;
+--------------------+---------------+------+-----+---------+-------+
| Field              | Type          | Null | Key | Default | Extra |
+--------------------+---------------+------+-----+---------+-------+
| SOURCE_NODE_ID     | decimal(18,0) | NO   | PRI |         |       |
| SOURCE_NODE_ENTITY | varchar(60)   | NO   | PRI |         |       |
| SINK_NODE_ID       | decimal(18,0) | NO   | PRI |         |       |
| SINK_NODE_ENTITY   | varchar(60)   | NO   | PRI |         |       |
| ASSOCIATION_TYPE   | varchar(60)   | NO   | PRI |         |       |
| SEQUENCE           | decimal(9,0)  | YES  |     | NULL    |       |
+--------------------+---------------+------+-----+---------+-------+

mysql> select distinct SOURCE_NODE_ENTITY from nodeassociation;
+--------------------+
| SOURCE_NODE_ENTITY |
+--------------------+
| Issue              |
| Project            |
+--------------------+

mysql> select distinct SINK_NODE_ENTITY from nodeassociation;
+-----------------------+
| SINK_NODE_ENTITY      |
+-----------------------+
| IssueSecurityScheme   |
| PermissionScheme      |
| IssueTypeScreenScheme |
| NotificationScheme    |
| ProjectCategory       |
| FieldLayoutScheme     |
| Component             |
| Version               |
+-----------------------+

mysql> select distinct ASSOCIATION_TYPE from nodeassociation;
+------------------+
| ASSOCIATION_TYPE |
+------------------+
| IssueVersion     |
| IssueFixVersion  |
| IssueComponent   |
| ProjectScheme    |
| ProjectCategory  |
+------------------+
```
So to get fix-for versions of an issue, run:
```
mysql> select * from projectversion where id in (
    select SINK_NODE_ID from nodeassociation where ASSOCIATION_TYPE='IssueFixVersion' and SOURCE_NODE_ID=(
        select id from jiraissue where pkey='JRA-5351')
    );
+-------+---------+-------+-------------+----------+----------+----------+------+-------------+
| ID    | PROJECT | vname | DESCRIPTION | SEQUENCE | RELEASED | ARCHIVED | URL  | RELEASEDATE |
+-------+---------+-------+-------------+----------+----------+----------+------+-------------+
| 11614 |   10240 | 3.6   | NULL        |      131 | NULL     | NULL     | NULL | NULL        |
+-------+---------+-------+-------------+----------+----------+----------+------+-------------+
```


Similarly with affects versions:

```
mysql> select * from projectversion where id in (
    select SINK_NODE_ID from nodeassociation where ASSOCIATION_TYPE='IssueVersion' and SOURCE_NODE_ID=(
        select id from jiraissue where pkey='JRA-5351')
    );
+-------+---------+---------------------+-------------+----------+----------+----------+------+---------------------+
| ID    | PROJECT | vname               | DESCRIPTION | SEQUENCE | RELEASED | ARCHIVED | URL  | RELEASEDATE         |
+-------+---------+---------------------+-------------+----------+----------+----------+------+---------------------+
| 10931 |   10240 |  3.0.3 Professional | NULL        |       73 | true     | NULL     | NULL | 2004-11-19 00:00:00 |
| 10930 |   10240 |  3.0.3 Standard     | NULL        |       72 | true     | NULL     | NULL | 2004-11-19 00:00:00 |
| 10932 |   10240 |  3.0.3 Enterprise   | NULL        |       74 | true     | NULL     | NULL | 2004-11-19 00:00:00 |
+-------+---------+---------------------+-------------+----------+----------+----------+------+---------------------+
```

and components:

```
mysql> select * from component where id in (
    select SINK_NODE_ID from nodeassociation where ASSOCIATION_TYPE='IssueComponent' and SOURCE_NODE_ID=(
        select id from jiraissue where pkey='JRA-5351')
    );
+-------+---------+---------------+-------------+------+------+--------------+
| ID    | PROJECT | cname         | description | URL  | LEAD | ASSIGNEETYPE |
+-------+---------+---------------+-------------+------+------+--------------+
| 10126 |   10240 | Web interface | NULL        | NULL | NULL |         NULL |
+-------+---------+---------------+-------------+------+------+--------------+
```





Issue links

```
mysql> desc issuelink;
+-------------+---------------+------+-----+---------+-------+
| Field       | Type          | Null | Key | Default | Extra |
+-------------+---------------+------+-----+---------+-------+
| ID          | decimal(18,0) | NO   | PRI |         |       |
| LINKTYPE    | decimal(18,0) | YES  | MUL | NULL    |       |
| SOURCE      | decimal(18,0) | YES  | MUL | NULL    |       |
| DESTINATION | decimal(18,0) | YES  | MUL | NULL    |       |
| SEQUENCE    | decimal(18,0) | YES  |     | NULL    |       |
+-------------+---------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

For instance, to list all links between TP-1 and TP-2:

```
mysql> select * from issuelink where SOURCE=(select id from jiraissue where pkey='TP-1') and DESTINATION=(select id from jiraissue where pkey='TP-2');
+-------+----------+--------+-------------+----------+
| ID    | LINKTYPE | SOURCE | DESTINATION | SEQUENCE |
+-------+----------+--------+-------------+----------+
| 10020 |    10000 |  10000 |       10010 |     NULL |
+-------+----------+--------+-------------+----------+
1 row in set (0.00 sec)
```
Link types are defined in issuelinktype. This query prints all links in the system with their type:

```
mysql> select j1.pkey, issuelinktype.INWARD, j2.pkey from jiraissue j1, issuelink, issuelinktype, jiraissue j2 where j1.id=issuelink.SOURCE and j2.id=issuelink.DESTINATION and issuelinktype.id=issuelink.linktype;
+-------+---------------------+-------+
| pkey  | INWARD              | pkey  |
+-------+---------------------+-------+
| TP-4  | jira_subtask_inward | TP-5  |
| TP-4  | jira_subtask_inward | TP-7  |
| TP-4  | jira_subtask_inward | TP-8  |
| TP-11 | jira_subtask_inward | TP-12 |
| TP-4  | jira_subtask_inward | TP-6  |
| TP-1  | is duplicated by    | TP-2  |
+-------+---------------------+-------+
6 rows in set (0.00 sec)
```


Subtasks
As shown in the last query, JIRA records the issue-subtask relation as a link. The "subtask" link type is hidden in the user interface (indicated by the 'pstyle' value below), but visible in the database:

```
mysql> select * from issuelinktype;
+-------+-------------------+---------------------+----------------------+--------------+
| ID    | LINKNAME          | INWARD              | OUTWARD              | pstyle       |
+-------+-------------------+---------------------+----------------------+--------------+
| 10000 | Duplicate         | is duplicated by    | duplicates           | NULL         |
| 10001 | jira_subtask_link | jira_subtask_inward | jira_subtask_outward | jira_subtask |
+-------+-------------------+---------------------+----------------------+--------------+
2 rows in set (0.00 sec)
```


Custom fields
 Custom fields defined in the system are stored in the customfield table, and instances of custom fields are stored in customfieldvalue: 
 

```
mysql> desc customfieldvalue;
+-------------+---------------+------+-----+---------+-------+
| Field       | Type          | Null | Key | Default | Extra |
+-------------+---------------+------+-----+---------+-------+
| ID          | decimal(18,0) | NO   | PRI |         |       |
| ISSUE       | decimal(18,0) | YES  | MUL | NULL    |       |
| CUSTOMFIELD | decimal(18,0) | YES  |     | NULL    |       |
| PARENTKEY   | varchar(255)  | YES  |     | NULL    |       |
| STRINGVALUE | varchar(255)  | YES  |     | NULL    |       |
| NUMBERVALUE | decimal(18,6) | YES  |     | NULL    |       |
| TEXTVALUE   | longtext      | YES  |     | NULL    |       |
| DATEVALUE   | datetime      | YES  |     | NULL    |       |
| VALUETYPE   | varchar(255)  | YES  |     | NULL    |       |
+-------------+---------------+------+-----+---------+-------+
```
We can print all custom field values for an issue with:

```
mysql> select * from customfieldvalue where issue=(select id from jiraissue where pkey='JRA-5448');
+-------+-------+-------------+-----------+-------------+-------------+-----------+---------------------+-----------+
| ID    | ISSUE | CUSTOMFIELD | PARENTKEY | STRINGVALUE | NUMBERVALUE | TEXTVALUE | DATEVALUE           | VALUETYPE |
+-------+-------+-------------+-----------+-------------+-------------+-----------+---------------------+-----------+
| 23276 | 22160 |       10190 | NULL      | NULL        |        NULL | NULL      | 2004-12-07 17:25:58 | NULL      |
+-------+-------+-------------+-----------+-------------+-------------+-----------+---------------------+-----------+
```
and we can see what type of custom field this (10190) is with:

```
mysql> select * from customfield where id=10190;
+-------+------------------------------------------------+--------------------------------------------------------+-----------------+-------------+--------------+-----------+---------+-----------+
| ID    | CUSTOMFIELDTYPEKEY                             | CUSTOMFIELDSEARCHERKEY                                 | cfname          | DESCRIPTION | defaultvalue | FIELDTYPE | PROJECT | ISSUETYPE |
+-------+------------------------------------------------+--------------------------------------------------------+-----------------+-------------+--------------+-----------+---------+-----------+
| 10190 | com.atlassian.jira.ext.charting:resolutiondate | com.atlassian.jira.ext.charting:resolutiondatesearcher | Resolution Date | NULL        | NULL         |      NULL |    NULL | NULL      |
+-------+------------------------------------------------+--------------------------------------------------------+-----------------+-------------+--------------+-----------+---------+-----------+
```
(ie. it's a "Resolution Date").

This query identifies a particular custom field value in a particular issue:

```
mysql> select stringvalue from customfieldvalue where customfield=(select id from customfield where cfname='Urgency') and issue=(select id from jiraissue where pkey='FOR-845');
+-------------+
| stringvalue |
+-------------+
| Low         | 
+-------------+
1 row in set (0.33 sec)
```

If the custom field has multiple values (multi-select or multi-user picker), each issue can have multiple customfieldvalue rows:

```
mysql> select * from customfieldvalue where customfield=(select ID from customfield where cfname='MultiUser');
+-------+-------+-------------+-----------+-------------+-------------+-----------+-----------+-----------+
| ID    | ISSUE | CUSTOMFIELD | PARENTKEY | STRINGVALUE | NUMBERVALUE | TEXTVALUE | DATEVALUE | VALUETYPE |
+-------+-------+-------------+-----------+-------------+-------------+-----------+-----------+-----------+
| 10002 | 10060 |       10000 | NULL      | bob         |        NULL | NULL      | NULL      | NULL      | 
| 10003 | 10060 |       10000 | NULL      | jeff        |        NULL | NULL      | NULL      | NULL      | 
+-------+-------+-------------+-----------+-------------+-------------+-----------+-----------+-----------+
2 rows in set (0.00 sec)
```
Here issue 10060 has two users, bob and jeff in its MultiUser custom field.




