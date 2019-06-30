

The PostgreSQL INSERT statement is used to insert a single record or multiple records 
into a table. More about PostgreSQL INSERT statement can be found here: https://www.postgresql.org/docs/11/sql-insert.html

## Insert statement

Insert example SQL table:
```sql
CREATE TABLE IF NOT EXISTS link (
    id serial PRIMARY KEY,
    url VARCHAR (255) NOT NULL,
    name VARCHAR (255) NOT NULL,
    description VARCHAR (255)
);
```

### Insert row by row

```
insertStmt := Link.INSERT(Link.ID, Link.URL, Link.Name, Link.Description).
    VALUES(100, "http://www.postgresqltutorial.com", "PostgreSQL Tutorial", DEFAULT).
    VALUES(101, "http://www.google.com", "Google", DEFAULT).
    VALUES(102, "http://www.yahoo.com", "Yahoo", nil)
```

There is also shorthand notation for inserting model data:
```
tutorial := model.Link{
    ID:   100,
    URL:  "http://www.postgresqltutorial.com",
    Name: "PostgreSQL Tutorial",
}

google := model.Link{
    ID:   101,
    URL:  "http://www.google.com",
    Name: "Google",
}

yahoo := model.Link{
    ID:   102,
    URL:  "http://www.yahoo.com",
    Name: "Yahoo",
}

insertStmt := Link.INSERT(Link.ID, Link.URL, Link.Name, Link.Description).
    MODEL(turorial).
    MODEL(google).
    MODEL(yahoo)
```
Or event shorter if model data is in the slice:
```
insertStmt := Link.INSERT(Link.ID, Link.URL, Link.Name, Link.Description).
    MODELS([]model.Link{turorial, google, yahoo})
```
`Link.ID, Link.URL, Link.Name, Link.Description` - is the same as Link.AllColumns 
so above statement can be simplified to:

```
insertStmt := Link.INSERT(Link.AllColumns).
    MODELS([]model.Link{turorial, google, yahoo})
```

`Link.ID` is a primary key autoincrement column so it can be omitted in INSERT statement.  
`Link.MutableColumns` - is shorthand notation for list of all columns minus primary key columns.

```
insertStmt := Link.INSERT(Link.MutableColumns).
    MODELS([]model.Link{turorial, google, yahoo})
```

Inserts using `VALUES`, `MODEL` and `MODELS` can appear as the part of the same insert statement.

```
insertStmt := Link.INSERT(Link.ID, Link.URL, Link.Name, Link.Description, Link.Description).
    VALUES(101, "http://www.google.com", "Google", DEFAULT, DEFAULT).
    MODEL(turorial).
    MODELS([]model.Link{yahoo})
``` 

### Insert using query
A query (SELECT statement) that supplies the rows to be inserted.
```
// duplicate first 10 entries
insertStmt := Link.
    INSERT(Link.URL, Link.Name).
    QUERY(
        SELECT(Link.URL, Link.Name).
            FROM(Link).
            WHERE(Link.ID.GT(Int(0)).AND(Link.ID.LT_EQ(10))),
    )
```

## Execute statement

To execute insert statement and get sql.Result:

```
res, err := insertStmt.Exec(db)
```

To execute insert statement and return row records inserted, insert statement has to have RETURNING clause:
```
insertStmt := Link.INSERT(Link.ID, Link.URL, Link.Name, Link.Description).
    VALUES(100, "http://www.postgresqltutorial.com", "PostgreSQL Tutorial", DEFAULT).
    VALUES(101, "http://www.google.com", "Google", DEFAULT).
    RETURNING(Link.ID, Link.URL, Link.Name, Link.Description)
    
dest := []model.Link{}

err := insertStmt.Query(db, &dest)
    
```

Use `ExecContext` and `QueryContext` to provide context object to execution.
