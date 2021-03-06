# pino-mysql
A [Pino Transport](https://getpino.io/#/docs/transports) for MySQL and MariaDB Databases

![pino-mysql](pino-mysql.png)


## Quick Start

This is a [transport](https://getpino.io/#/docs/transports) for the
wonderful [Pino](https://getpino.io) logger.

A "transport" just means that it is an application that can take
(transport) your logs to a destination source. In this case, we can
save your logs into a [MySql](https://www.mysql.com/) (or [MariaDB](https://mariadb.org/)) database. This is done in three easy steps:

1. Create your database table to save the logs: `CREATE TABLE logs (log JSON);`
2. Create a configuration file that tells us db connection and table
   details: `db-configuration.json`
3. Pipe the log output and watch it pour into your database:
    `run <my-app> | pino-mysql -c db-configuration.json`


## Using pipes

We pipe the log output as recommended by [Pino](https://getpino.io) so
that it does not interfere with the running of the application.

`pino-mysql` will echo the logstream by default allowing us to chain
pipes so we can do nice things like:

```
    run <my-app> | pino-mysql -c db-configuration.json | pino-pretty
```

Use the `-q|--quiet` configuration switch if you do not want this
behaviour.


## The database configuration file
The configuration file is a JSON file with fields like this:

```
{
    "host"     : "localhost",
    "user"     : "me",
    "password" : "secret",
    "database" : "my_db",
    "table"    : "logs",
    "columns"  : {
        "log"  : "*",
        "name" : "name",
        "unix" : "time",
        "url"  : "req.url"
    }
}
```

Create a table to save this that looks like this:

```
CREATE TABLE logs (
    log_id INT NOT NULL AUTO_INCREMENT,
    log JSON,
    unix BIGINT UNSIGNED,
    name VARCHAR(64),
    url VARCHAR(64),

    PRIMARY KEY ( log_id )
)
```



## Mapping log data to Database columns

You can save the entire log (as JSON) by using `"*"` against the column
name. When you do this I highly recommend using the new [JSON field type](https://dev.mysql.com/doc/refman/8.0/en/json.html)
to save your JSON. It will save in an efficient binary format _and_
allow you to query your JSON directly in SQL. Amazing!

Instead if you are only interested in particular fields, I've got your
back there too! `pino-mysql` will extract and save particular log fields
into the columns (by using the field paths `"name"`, `"req.url"`, etc).
You can see examples of this in the definition of "url" above.

### Using a custom delimiter

If you have a dotted field in your log input that you would like to
parse, you can use provide a custom delimiter in the configuration JSON:

```
...
    "table"    : "logs",
    "columns"  : {
        ...
        "url"  : "req-url",
        "xid"  : "dotted.field-subfield"
    },
    "delimiter": "-"
}
```

### Using MySQL Generated Columns with JSON

Mysql also has the ability to extract fields and if you prefer it done
by the DB you can define a
['generated
column'](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)
that extracts the given field instead:

```
    req_url VARCHAR(128) GENERATED ALWAYS as ( json_unquote(json_extract(log, '$.req.url')) ) STORED

```


## Using npx

Once installed, using this transport is simply:

```
    run <my-app> | pino-mysql -c db-configuration.json
```

If you prefer using
[npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)
that works as well:

```
    run <my-app> | npx pino-mysql -c db-configuration.json
```


### HTH & Enjoy! :-)


