# [https://docs.splunk.com/Documentation/Splunk/7.1.0/SearchReference/Rex](https://docs.splunk.com/Documentation/Splunk/7.1.0/SearchReference/Rex)

# rex

## Description

Use this command to either extract fields using regular expression named groups, or replace or substitute characters in a field using sed expressions.

Use the rex command for search-time field extraction or string replacement and character substitution.

## Syntax

```
rex [field=<field>] ( <regex-expression> [max_match=<int>] [offset_field=<string>] ) | (mode=sed <sed-expression>)
```

## Examples

### Example 1:

Extract "from" and "to" fields using regular expressions. If a raw event contains "From: Susan To: Bob", then`from=Susan`and`to=Bob`.

`... | rex field=_raw "From: (?<from>.*) To: (?<to>.*)"`

### Example 2:

Extract "user", "app" and "SavedSearchName" from a field called "savedsearch\_id" in scheduler.log events. If`savedsearch_id=bob;search;my_saved_search`then`user=bob`,`app=search`and`SavedSearchName=my_saved_search`

`... | rex field=savedsearch_id "(?<user>\w+);(?<app>\w+);(?<SavedSearchName>\w+)"`

### Example 3:

Use`sed`syntax to match the regex to a series of numbers and replace them with an anonymized string.

`... | rex field=ccnumber mode=sed "s/(\d{4}-){3}/XXXX-XXXX-XXXX-/g"`

### Example 4:

Display IP address and ports of potential attackers.

`sourcetype=linux_secure port "failed password" | rex "\s+(?<ports>port \d+)" | top src_ip ports showperc=0`

This search used rex to extract the port field and values. Then, it displays a table of the top source IP addresses \(src\_ip\) and ports the returned with the search for potential attackers.

