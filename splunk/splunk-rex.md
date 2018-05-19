# https://docs.splunk.com/Documentation/Splunk/7.1.0/SearchReference/Rex

# rex

## Description

Use this command to either extract fields using regular expression named groups, or replace or substitute characters in a field using sed expressions.

Use the rex command for search-time field extraction or string replacement and character substitution.

## Syntax

```
rex [field=<field>] ( <regex-expression> [max_match=<int>] [offset_field=<string>] ) | (mode=sed <sed-expression>)
```



