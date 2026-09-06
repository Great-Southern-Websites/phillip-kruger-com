---
title: "The Matrix"
slug: the-matrix
description: "Java Community Keynote The last day of the conference started, as usual, with the Java Community Keynote. Some of the highlights:"
thumb: site/javaone_logo.jpg
date: 2017-10-05
tags: ["JavaOne 2017"]
---

## Java Community Keynote

The last day of the conference started, as usual, with the Java Community Keynote. Some of the highlights:

<img src="{=site.url('images/posts/the-matrix/ibm.jpg')}" alt="">

- IBM Open Sourced their JDK (Open J9), their Application server (OpenLiberty).
- Quick highlight on some of the things we now have in Microprofile:
  - `@Retry(maxRetries = 2)`
  - `@CircuitBreaker(requestVolumeThreshold=3, failureRate=0.5, delay=1000)`
  - `@Fallback(SimpleFallbackHandler.class)`

- IBM Private Cloud built on Kubernetes
- Java EE (now) and Microprofile at Eclipse Foundation
- Kubernetes deploying and clustering over Google Cloud, IBM Cloud and Oracle Cloud

<img src="{=site.url('images/posts/the-matrix/mp.jpg')}" alt="">

The very funny version of “The Matrix” took us out.

<img src="{=site.url('images/posts/the-matrix/matrix.jpg')}" alt="">

## Dos and Don’ts of JSON Processing in a Database

Very interesting talk on storing and processing JSON in the Database. Of course you rather want to use a Document store, Key-value store or Search Engine than storing JSON in a relational database.

But in cases that you have a relational DB and want to store JSON in a table, look at [Oracle SODA](https://docs.oracle.com/cd/E56351_01/doc.30/e58123/rest.htm#ADRST107).

You can do SQL Operations (Mixed with your normal Table search) on this JSON

```sql
{|SELECT json_value(json_document FORMAT JSON, '$.Reference')
  FROM "MyCollection" WHERE id = 'identifier';|}
```

## Why You’re Going to Fail Running Java on Docker!

<img src="{=site.url('images/posts/the-matrix/fail.jpg')}" alt="">

This great talk by [Burr Sutter](https://twitter.com/burrsutter) and [Rafael Benevides](https://twitter.com/rafabene) from [RedHat](https://twitter.com/RedHatNews), showed some pitfalls you need to look out for, specifically the Memory and other hardware settings that the JVM will read wrong. This is not JVM specific, other Linux commands are also not updated to understand containerization, example `free`

You can see the presentation [here](http://bit.ly/javadockerfail)

## Migrating to Java 9 Modules

<img src="{=site.url('images/posts/the-matrix/j9modules.jpg')}" alt="">

[Paul Bakker](https://twitter.com/pbakker) from Netflix gave a practical talk on how to migrate from older Java versions to Java 9. You can find the sildes [here](https://t.co/W8mpLJlTHi). He and [Sander Mak](https://twitter.com/Sander_Mak) just released a book called [Java 9 Modularity](https://twitter.com/javamodularity), that you can buy [here](https://javamodularity.com/)

There are a few caveats to look out for, and ways to get around them. Even if your app’s dependencies have not yet migrated to Java 9, you can sill use them as modules.

- `java --add-modules`
- Automatic Modules
- `java --add-opens`
- Maven already support Java 9

## Twelve Ways to Make Code Suck Less

<img src="{=site.url('images/posts/the-matrix/12ways.jpg')}" alt="">

One of 7 talks that [Venkat Subramaniam](https://twitter.com/venkat_s) gave at JavaOne. His talks are always good en very entertaining.

*“We can not be agile if our code sucks”*

1 Technical Debt 1 Higher Cohesion 1 Loose Coupling 1 Program with Intention 1 Avoid Primitive Obsession 1 Prefer clean code over clever code 1 [Zinssar’s Principle](https://lonetechnicalwriter.wordpress.com/2015/05/20/4-principles-of-writing/) 1 Comment why not what 1 Avoid long methods, apply SLAP 1 Give good meaningful names 1 No tactical code reviews 1 Reduce state & state mutation

## The Basics of Machine Learning

Machine Learning is an important part of Artificial Intelligence that learn from experience to make predictions.

<img src="{=site.url('images/posts/the-matrix/ml.jpg')}" alt="">

Due to the improved tech and the price of storage, and the fact that the environment needs it, Machine Learning is very popular now.

## The fat lady sings.

End of yet another awesome JavaOne.

<img src="{=site.url('images/posts/the-matrix/swag.jpg')}" alt="">

5 days, 4 party, 3 keynotes, 33 sessions, many new friends and all this swag.
