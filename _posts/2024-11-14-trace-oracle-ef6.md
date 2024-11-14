---
layout: post
title: Tracing Oracle.ManagedDataAcces.Client queries
date: 2024-11-14
categories: oracle csharp ef6
---

## Why tracing?

In the project I am currently working on, we connect to the Oracle database from the .Net Framework and WPF application level. We use the Oracle.ManagedDataAccess.Client package.

Unfortunately, in the case of an incorrect query, Oracle throws an exception that does not say much. I also did not find another way to find out what query the Entity Framework creates for the Oracle database.

## Snippet

A small piece of code that will allow you to enable tracking between Oracle.ManagedDataAccess.Client and the database in app.config

```xml

<oracle.manageddataaccess.client>
    <version number="*">     
        <settings>              
             <setting name="TraceLevel" value="7" />
             <setting name="TraceFileLocation" value="c:\temp\"/>
        </settings> 
    </version>
</oracle.manageddataaccess.client>

```




