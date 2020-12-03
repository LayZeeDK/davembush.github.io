---
title: SQL For Programmers - Finding a String
tags:
  - like
  - sql
  - tsql
  - where clause
url: 339.html
id: 339
categories:
  - SQL For Programmers
date: 2013-12-04 18:27:40
---

![tp_vol3_025](/uploads/2008/09/tp-vol3-025.jpg) Many times in our queries, we aren't looking for an exact match.  We are looking for one string that exists in another.  There are a couple statements available to us that will allow us to do this. The first of these is the LIKE statement, and if you are familiar with DOS or the Linux equivalent, this should look familiar to you.  The most common usage of LIKE looks like this:

<!-- more -->

``` sql
SELECT * FROM someTable
WHERE fieldName LIKE '%stringHere%'
```

Note the % inside the single quote marks. This statement is telling SQL to find all of the rows from someTable where the contents of fieldName has the string 'stringHere' in it. You could also do something like:

``` sql
SELECT * FROM someTable
WHERE fieldName LIKE 'stringHere%'
```

which would find all the rows where fieldName started with 'stringHere' You can also use the following pattern matching characters:

Symbol

Descriptions

_

Look for any single character

``` sql
[]
```

Look for any single character listed in the set fieldName LIKE '\[abc\]%' Looks for anything that starts with a, b, or c.

``` sql
[^]
```

Look for any single character not listed in the set fieldName LIKE `'[^abc]%'` Looks for anything that doesn't start with a, b or c.

If you need to find a string that includes one of these symbols, you'll need to ESCAPE it and you'll need to define the escape sequence. So to look for a % in the middle of the string you would use:

``` sql
SELECT * FROM someTable
WHERE fieldName LIKE '%%stringHere%' ESCAPE '%'
```

This will look for all rows where the fieldName starts with the string `'%stringHere'` because we've told it that `%` is the escape character and we've used `%%` twice in our search string.