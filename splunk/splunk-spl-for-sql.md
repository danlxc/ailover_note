## From SQL to Splunk SPL

SQL is designed to search relational database tables which are comprised of**columns**. SPL is designed to search events, which are comprised of**fields**. In SQL, you often see examples that use "mytable" and "mycolumn". In SPL, you will see examples that refer to "fields". In these examples, the "source" field is used as a proxy for "table". In Splunk software, "source" is the name of the file, stream, or other input from which a particular piece of data originates, for example`/var/log/messages`or`UDP:514`.

When translating from any language to another, often the translation is longer because of idioms in the original language. Some of the Splunk search examples shown below could be more concise, but for parallelism and clarity, the SPL table and field names are kept the same as the SQL example.

* SPL searches rarely need the FIELDS command to filter out columns because the user interface provides a more convenient method for filtering. The FIELDS command is used in the SPL examples for parallelism.
* With SPL, you never have to use the AND operator in Boolean searches, because AND is implied between terms. However when you use the AND or OR operators, they must be specified in uppercase.
* SPL commands do not need to be specified in uppercase. In the these SPL examples, the commands are specified in uppercase for easier identification and clarity.

| SQL command | SQL example | Splunk SPL example |
| :--- | :--- | :--- |
| SELECT \* | SELECT \*FROM mytable | source=mytable |
| WHERE | SELECT \*FROM mytableWHERE mycolumn=5 | source=mytable mycolumn=5 |
| SELECT | SELECT mycolumn1, mycolumn2FROM mytable | source=mytable	\| FIELDS mycolumn1, mycolumn2 |
| AND/OR | SELECT \*FROM mytableWHERE \(mycolumn1="true"   OR mycolumn2="red"\) AND mycolumn3="blue" | source=mytableAND \(mycolumn1="true"   OR mycolumn2="red"\)AND mycolumn3="blue"**Note:**The AND operator is implied in SPL and does not need to be specified. For this example you could also use:source=mytable\(mycolumn1="true"   OR mycolumn2="red"\)mycolumn3="blue" |
| AS \(alias\) | SELECT mycolumn AS column\_aliasFROM mytable | source=mytable\| RENAME mycolumn as column\_alias\| FIELDS column\_alias |
| BETWEEN | SELECT \*FROM mytableWHERE mycolumnBETWEEN 1 AND 5 | source=mytable   mycolumn&gt;=1 mycolumn&lt;=5 |
| GROUP BY | SELECT mycolumn, avg\(mycolumn\)FROM mytableWHERE mycolumn=valueGROUP BY mycolumn | source=mytable mycolumn=value\| STATS avg\(mycolumn\) BY mycolumn\| FIELDS mycolumn, avg\(mycolumn\)Several commands use a`by-clause`to group information, including[chart](http://docs.splunk.com/Documentation/Splunk/6.4.3/SearchReference/Chart),[rare](http://docs.splunk.com/Documentation/Splunk/6.4.3/SearchReference/Rare),[sort](http://docs.splunk.com/Documentation/Splunk/6.4.3/SearchReference/Sort),[stats](http://docs.splunk.com/Documentation/Splunk/6.4.3/SearchReference/Stats), and[timechart](http://docs.splunk.com/Documentation/Splunk/6.4.3/SearchReference/Timechart). |
| HAVING | SELECT mycolumn, avg\(mycolumn\)FROM mytableWHERE mycolumn=valueGROUP BY mycolumnHAVING avg\(mycolumn\)=value | source=mytable mycolumn=value\| STATS avg\(mycolumn\) BY mycolumn\| SEARCH avg\(mycolumn\)=value\| FIELDS mycolumn, avg\(mycolumn\) |
| LIKE | SELECT \*FROM mytableWHERE mycolumn LIKE "%some text%" | source=mytable   mycolumn="\*some text\*"**Note:**The most common search in Splunk SPL is nearly impossible in SQL - to search all fields for a substring. The following SPL search returns all rows that contain "some text" anywhere:source=mytable "some text"  |
| ORDER BY | SELECT \*FROM mytableORDER BY mycolumn desc | source=mytable\| SORT -mycolumnIn SPL you use a negative sign \( - \) in front of a field name to sort in descending order. |
| SELECT DISTINCT | SELECT DISTINCT   mycolumn1, mycolumn2FROM mytable | source=mytable\| DEDUP mycolumn1\| FIELDS mycolumn1, mycolumn2 |
| SELECT TOP | SELECT TOP\(5\) mycolum1, mycolum2FROM mytable1WHERE mycolum3 = "bar"ORDER BY mycolum1 mycolum2 | Source=mytable1 mycolum3="bar"\| FIELDS mycolum1 mycolum2\| SORT mycolum1 mycolum2\| HEAD 5 |
| INNER JOIN | SELECT \*FROM mytable1INNER JOIN mytable2ON mytable1.mycolumn=   mytable2.mycolumn | source=mytable1\| JOIN type=inner mycolumn   \[SEARCH source=mytable2\]**Note:**There are two other methods to join tables:Use the`lookup`command to add fields from an external table:... \| LOOKUP myvaluelookup   mycolumn   OUTPUT myoutputcolumnUse a subsearch:source=mytable1  \[SEARCH source=mytable2     mycolumn2=myvalue    \| FIELDS mycolumn2\]If the columns that you want to join on have different names, use the`rename`command to rename one of the columns. For example, to rename the column in mytable2:source=mytable1 \| JOIN type=inner mycolumn   \[ SEARCH source=mytable2     \| RENAME mycolumn2     AS mycolumn\]To rename the column in mytable1:source=mytable1 \| RENAME mycolumn1 AS mycolumn \| JOIN type=inner mycolumn   \[SEARCH source=mytable2\]You can rename a column regardless of whether you use the JOIN command, a lookup, or a subsearch. |
| LEFT \(OUTER\) JOIN | SELECT \*FROM mytable1LEFT JOIN mytable2ON mytable1.mycolumn=  mytable2.mycolumn | source=mytable1\| JOIN type=left mycolumn   \[SEARCH source=mytable2\] |
| SELECT INTO | SELECT \*INTO new\_mytable IN mydb2FROM old\_mytable | source=old\_mytable\| EVAL source=new\_mytable\| COLLECT index=mydb2**Note:**COLLECT is typically used to store expensively calculated fields back into your Splunk deployment so that future access is much faster. This current example is atypical but shown for comparison to the SQL command. The source will be renamed orig\_source |
| TRUNCATE TABLE | TRUNCATE TABLE mytable | source=mytable\| DELETE |
| INSERT INTO | INSERT INTO mytableVALUES \(value1, value2, value3,....\) | **Note:**see SELECT INTO. Individual records are not added via the search language, but can be added via the API if need be. |
| UNION | SELECT mycolumnFROM mytable1UNIONSELECT mycolumn FROM mytable2 | source=mytable1\| APPEND   \[SEARCH source=mytable2\]\| DEDUP mycolumn |
| UNION ALL | SELECT \*FROM mytable1UNION ALLSELECT \* FROM mytable2 | source=mytable1\| APPEND   \[SEARCH source=mytable2\]  |
| DELETE | DELETE FROM mytableWHERE mycolumn=5 | source=mytable1 mycolumn=5\| DELETE |
| UPDATE | UPDATE mytableSET column1=value,   column2=value,...WHERE some\_column=some\_value | **Note:**There are a few things to think about when updating records in Splunk Enterprise. First, you can just add the new values to your Splunk deployment \(see INSERT INTO\) and not worry about deleting the old values, because Splunk software always returns the most recent results first. Second, on retrieval, you can always de-duplicate the results to ensure only the latest values are used \(see SELECT DISTINCT\). Finally, you can actually delete the old records \(see DELETE\). |



