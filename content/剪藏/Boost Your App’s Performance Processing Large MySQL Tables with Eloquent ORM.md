---
title: "Boost Your App’s Performance: Processing Large MySQL Tables with Eloquent ORM"
source: https://medium.com/@JoseCardona/boost-your-apps-performance-processing-large-mysql-tables-with-eloquent-orm-3724421d97aa
author:
  - "[[Jose Manuel Cardona]]"
published: 2023-08-07
created: 2025-06-30
description: In this post we’ll explore how to leverage cursors and unbuffered queries to unlock the speed and scalability of Eloquent for giant datasets. Eloquent ORM provides a powerful ActiveRecord…
---

In this post we’ll explore how to leverage cursors and unbuffered queries to unlock the speed and scalability of Eloquent for giant datasets.  

## The Basics of Eloquent ORM Cursors

Eloquent ORM provides a powerful ActiveRecord implementation for working with your MySQL database in Laravel. However, when processing large numbers of records, Eloquent can become slow and consume substantial amounts of memory.

This is because Eloquent instantiates a full model object for each row by default. With thousands of rows, this creates huge memory overhead.

Fortunately, Eloquent offers a `[cursor](https://laravel.com/docs/9.x/queries#cursor-pagination)` method that allows efficiently iterating through a large result set without instantiating a model for each record.

The `cursor` returns a generator that lets you loop through the results and process each record without loading it into memory:

```c
foreach (Book::where('category', 'Fiction')->cursor() as $book) {
  // Process each book by creating a model instance only for the current iteration
}
```

Using the `cursor` allows you to take action on each record while avoiding the memory overhead of loading all model instances upfront. Instead, it creates a model instance only for each iteration of the loop. This keeps memory usage low while still leveraging Eloquent models for convenient data access. This can provide a significant performance boost over loading full models.

## Unbuffered Queries for Faster Processing

While the Eloquent cursor helps avoid excessive memory usage, we can take optimization further by configuring [**unbuffered queries**](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php) at the database level.

By default, MySQL buffers the entire query result set before returning any rows. With unbuffered queries, MySQL returns rows as soon as they are found, providing much faster start times for large datasets.

To use unbuffered queries in Laravel, it’s best to create a separate database connection:

```c
// In database.php
'connections' => [

    'mysql' => [
        // Default buffered connection
    ],

    'mysql_unbuffered' => [
        // Unbuffered connection
        'driver' => 'mysql',
        'database'  => env('DB_DATABASE'),
        'options' => [
            PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false,
        ]
    ]

]
```

This isolates unbuffered queries from your main connection. Results start streaming immediately without having to wait for the full buffer.

We can also configure it at runtime:

```c
DB::connection()->getPdo()->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);
```

However, some limitations exist:

- Only one unbuffered query can run at a time on a connection.
- The unbuffered result set must be kept active until finished, or `MySQL has gone away` errors can occur.  

## Avoid Offset/Limit Pagination for Large Datasets

A common mistake when processing large MySQL tables is using offset and limit to paginate through the results. This pagination approach can lead to slow performance at scale.

The issue is that offset forces MySQL to scan over a large number of rows before returning results. The larger the offset, the more rows it has to skip first.  

For example, fetching `LIMIT 100 OFFSET 50000` would require MySQL to scan over 50,000 rows before returning the first result. For big tables, this results in very slow query times.

Instead, cursors with unbuffered queries start returning results immediately. It only keeps a small buffer in memory as it streams rows. This provides much faster response times.

So while offset/limit pagination works for small datasets, it should be avoided for large tables. The cursor method combined with unbuffered queries is far more efficient at scale.  

## Putting It All Together: Cursors with Unbuffered Queries

Now that we’ve covered both Eloquent cursors and unbuffered queries independently, let’s look at how to use them together for optimized performance.

First, we’ll configure an unbuffered database connection in database.php:

```c
'connections' => [
    //...
    'mysql_unbuffered' => [
        'driver' => 'mysql',
        'database'  => env('DB_DATABASE'),
        'options' => [
            PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false, 
        ]
    ]
]
```

Then we can leverage the model with the cursor and unbuffered connection:

```c
use App\Models\Book;

Book::on('mysql_unbuffered')
    ->where('category', 'Fiction')
    ->cursor()
    ->each(function ($book) {
        // Process each book
    });
```

This gives us streaming, memory efficient results. The query starts fast, and rows process quickly for processing big tables!

We can take this further by chunking the cursor results:

```c
use App\Models\Book;

Book::on('mysql_unbuffered')
    ->where('category', 'Fiction')
    ->cursor() 
    ->chunk(100, function ($books) {
        // Process each chunk of 100 books
    });
```

Now we get batched processing in addition to fast streaming! This allows efficiently working with giant tables while controlling memory usage.

## Other Considerations and Best Practices

While Eloquent cursors and unbuffered queries can provide significant performance improvements, there are other optimizations to consider when working with large MySQL tables:

**Index Tables for Faster Lookups**

Adding appropriate indexes on columns used for lookups and joins can further speed up query performance. This enhances the benefits of unbuffered queries.

**Process in Batches for Memory Control**

Rather than processing an entire huge result set at once, work in smaller batched chunks. This provides more control over memory usage.

**Cursors Do Not Work Well with Aggregate Methods**

Because cursors use generators, if you call aggregate methods like count(), sum(), min(), max(), etc. directly on a cursor it would require loading the entire result set into memory to calculate.

Methods like these require access to all rows, so they are incompatible with this strategy. This is an important consideration when deciding whether to use a cursor.

**Benchmark Improvements with Real Data**

When testing optimizations, use a copy of real production data rather than synthetic test data. This provides a more accurate benchmark of improvements.  

By combining these best practices with Eloquent cursors and unbuffered queries, you can unlock maximum performance and scalability. The optimizations work together to make processing huge MySQL tables fast and efficient.

## Conclusion

Eloquent ORM can become slow and memory intensive when processing large MySQL tables, due to loading full model objects.

However, Eloquent cursors allow efficiently looping through results without instantiating models. Unbuffered queries return rows faster by avoiding table scans.

Combining these two optimizations provides significant performance gains through faster processing and lower memory usage. Queries start faster and rows stream efficiently.

Additional best practices like indexing, and batch processing further improve speed and scalability when handling big datasets with Eloquent.

With the right techniques, you can leverage Eloquent to efficiently process millions of rows. Optimizations unlock the full power of Eloquent ORM for large MySQL tables.