# POSTGRESQL

## MySQL vs PostgreSQL

### MySQL
* focus on speed
* fast in simple operations (queries)
* MyISAM (storage engine) performs faster (simple queries like count(*)) but at the cost of not supporting the following:
  * cache the row count information
  * transactions
  * table locking
  * foreign keys
  * not guaranteed data durability
* InnoDB (storage engine; current engine that we are using)
  * ACID compliant (Atomicity, Consistency, Isolation, Durability)
  * modern choice (of engine)
  * [but using InnoDB in MySQL fails ACIDity](http://www.wikivs.com/wiki/MySQL_vs_PostgreSQL#ACID_Compliance)
* supports Native Asynchronous I/O for Linux
* subquery and joins support is more recent
  * performance is still behind that of PostgreSQL
  * [cases where InnoDB join will crash instead of being slow](http://www.wikivs.com/wiki/MySQL_vs_PostgreSQL#Joins)
  * NOT IN do not work the same across DBMS
* [open-source PRODUCT](http://www.wikivs.com/wiki/MySQL_vs_PostgreSQL#Development)

### PostgreSQL
* focus on features and standards (Full-text search, Schema for multitenancy)
* more reliable and faster in complex operations (subqueries or joins)
* scale better with large numbers of cores
* fully ACID compliant
  * more rigorous approach to robustness and data integrity
* supports full fledged asynchronous API (increase performance by up to 40%)
* IP Address datatypes (supports a range of IP-related functions)
* subquery already been optimised
* has PgAgent for scheduling
* [open-source PROJECT](http://www.wikivs.com/wiki/MySQL_vs_PostgreSQL#Development)
* not available to windows
* stricter and reliable (like that of Oracle)
* conform to existing database standards


## [Moving from MySQL to PostgreSQL](https://wiki.postgresql.org/wiki/Things_to_find_out_about_when_moving_from_MySQL_to_PostgreSQL)
1.  Comments:
    * MySQL - nonstandard `#`
    * PostgreSQL - ANSI standard double dash `--`
2.  Double qoute `"` or Single qoute `'`
    * MySQL
      * nonstandard; uses `'` or `"` to quote values `WHERE name = "John"` is the same as `WHERE name = 'John'`
      * uses backtick or accent mark "`" to quoute identifiers
    * PostgreSQL
      * uses `'` for quote values `WHERE name = 'John'`
      * uses `"` for identifiers (like field names, table names) `WHERE "name" = 'John'`
3.  String comparisons
    * MySQL - not case sensitive
    * PostgreSQL - case sensisitive
      * use conversion function like `lower()`
      * use case-insensitive operator like `ILIKE` or `~*`
4. Date handling is different for both
5. Operators
   * MySQL - uses C-language operators for logic (nonstandard)
     * `||` is the same as `OR`
     * `&&` is the same as `AND`
   * PostgresQL
     * `||` is for string concatenation
    
## Migrating MySQL Database to PostgreSQL

#### MySQL Dump

```
 mysqldump -uroot -p --compatible=postgresql --no-create-info --fields-optionally-enclosed-by=0x22 --default-character-set=utf8 --fields-terminated-by='|' --fields-escaped-by='\' -T /tmp [database name]
```

* `fields-terminated-by='|'` - for blobs that have `,` values
* `fields-optionally-enclosed-by=0x22` - for blobs that have `\r\n` values


#### PostgreSQL Import
1. Using `COPY`
 
  ```
   COPY [table name] FROM '[file]' WITH CSV DELIMITER '|' NULL '\N' ESCAPE '\'
  ```

2. Using `psql`

  ```
   psql [database name] < [file] [username]
  ```

## Short Guide for PostgreSQL
1.  To access PostgreSQL, `psql [db name] [username]`
2.  To exit, `\q`
3.  To list databases, `\l` or `\list`
4.  To list schemas (tenants), `\dn`
5.  To list tables (on the current schema, `public` by default), `\dt`
6.  To describe a table, `\d [table name]`
7.  `\?` for help
8.  To restart PostgreSQL daemon, `service postgresql restart`


## Gotchas in PostgreSQL
1. PostgreSQL Users
 * modify `pg_hba.conf` if needed to change auth-method
  * `md5` - require to supply an MD5-encrypted password for authentication (e.g. root in mysql)
  * `peer` - obtain the client's os user name; only available for local connections (e.g. deployuser in servers)
 * for a dedicated database server, look for the line
 
 ```
  # Put your actual configuration here
  # ..
 ```

 and append the following
    
 ```
  # TYPE  DATABASE  USER  ADDRESS    METHOD
  host    all       all   0.0.0.0/0  md5
 ```

2. include `-O` (no-owner) switch when dumping

 ```
  pg_dump -U [username] [db name] -f [file] -O
 ```

3. when setting up a separate slice for your PostgreSQL, make sure to install `libpg-dev` or your `gem install pg` will fail
4. can not drop database if a process is using it (including rails server, rails console, nginx)
5. when dumping a database from MySQL, need to update `[table_name]_id_seq`, e.g. for users table

 ```
  SELECT setval('users_id_seq', (SELECT MAX(id) FROM users));
 ```

6. sorting! - depending on the collation (e.g. `en_PH.utf8, en_US.utf8, C, POSIX, C.UTF-8, ucs_basic`) of the database (default value from template database)

 ```
  name = Server 01, Server 03, Server01, Server02, Server03, server 01, server 02, server 03 ,server01, server03
 
  # when using en_PH.utf8 or en_US.utf8
  # ORDER BY name ASC
  server01
  server 01
  Server01
  Server 01
  server 02
  Server02
  server03
  server 03
  Server03
  Server 03
 
  # when using C or C.UTF-8
  # ORDER BY name ASC
  Server 01
  Server 03
  Server01
  Server02
  Server03
  server 01
  server 02
  server 03
  server01
  server03
 ```

   * work around to fix sorting (C or C.UTF-8, one site said ucs_basic) is to update the datcollate column of the database (not fully tested for other unexpected behavior)

 ```
  UPDATE pg_database SET datcollate = '[collation]' WHERE datname = '[db name]';
 ```

   * to list available collation

 ```
  SELECT * FROM pg_collation;
 ```


##### Resources
1. http://www.wikivs.com/wiki/MySQL_vs_PostgreSQL
2. https://wiki.postgresql.org/wiki/Why_PostgreSQL_Instead_of_MySQL:_Comparing_Reliability_and_Speed_in_2007
3. https://wiki.postgresql.org/wiki/Things_to_find_out_about_when_moving_from_MySQL_to_PostgreSQL
4. http://posulliv.github.io/2012/06/29/mysql-postgres-bench/
5. http://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html
6. http://postgresql.1045698.n5.nabble.com/a-strange-order-by-behavior-td4513038.html
7. http://stackoverflow.com/questions/5090858/how-do-you-change-the-character-encoding-of-a-postgres-database
8. http://stackoverflow.com/a/5090923
9. https://www.digitalocean.com/community/articles/scaling-ruby-on-rails-setting-up-a-dedicated-postgresql-server-part-3
