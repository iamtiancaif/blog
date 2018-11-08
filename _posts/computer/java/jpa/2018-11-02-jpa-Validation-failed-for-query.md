---
layout: post
title: "jpa Validation failed for query"
categories: computer
tags: computer java jpa stackoverflow
author: web
source: 'https://stackoverflow.com/questions/44647630/validation-failed-for-query-for-method-jpql'
---

* content
{:toc}


You have tell Spring to treat that query as native one. Otherwise it will try to validate it according to the JPA specification.

Try:

    @Query(value = "SELECT ...", nativeQuery = true)
    public List<Object[]> transactions();

Keep in mind that cannot use the NEW operator syntax in this case and you would have to deal with result as an array of `Object`.

**Alternatively**

If you want to use map the results directly to a POJO class you would have to (assuming that you are using JPA 2.1+):

**1)** Define the mapping:

    @SqlResultSetMapping(
        name="transactionsMapping",
        classes={
            @ConstructorResult(
                targetClass=ConsolidateResDB.class,
                columns={
                    @ColumnResult(name="transdate"),
                    @ColumnResult(name="orderreqid")
                    // further mappings ...
                }
            )
        }
    )

**2)** Define a native query

    @NamedNativeQuery(name="transactions"
        , query="SELECT DATE_FORMAT(ts, '%d-%m-%Y') AS transdate, IFNULL(COUNT(orderreqid),0) ... ")

**3)** Define this method in the `CrudRepository` without the `@Query` annotation:

    public List<ConsolidateResDB> transactions();


