# sqltocsv

A library designed to let you easily turn any arbitrary sql.Rows result from a query into a CSV file with a minimum of fuss.

This is very much a work in progress at this stage.

## Usage

Importing the package

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql" // or the driver of your choice
    "github.com/joho/sqltocsv"
)
```

Dumping a query to a file

```go
// we're assuming you've setup your sql.DB etc elsewhere
rows, _ := db.Query("SELECT * FROM users WHERE something=72")

err := sqltocsv.WriteFile("~/important_user_report.csv", rows)
if err != nil {
    panic(err)
}
```

Return a query as a CSV download on the world wide web

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    rows, _ := db.Query("SELECT * FROM users WHERE something=72")

    w.Header().Set("Content-type", "text/csv")
    w.Header().Set("Content-Disposition", "attachment; filename=\"important_user_repost.csv\"")

    sqltocsv.Write(w, rows)
})
http.ListenAndServe(":8080", nil)
```

`Write` and `WriteFile` should do cover the common cases by the power of _Sensible Defaults™_ but if you need more flexibility you can get an instance of a `Converter` and fiddle with a few settings.

```go
rows, _ := db.Query("SELECT * FROM users WHERE something=72")

csvConverter := sqltocsv.New(rows)

csvConverter.DateTimeFormat = time.RFC822
csvConverter.Headers = append(rows.Columns(), "extra_column_one", "extra_column_two")

csvConverter.SetRowPreProcessor(func (columns []string) (bool, []string) {
    // exclude admins from report
    // NOTE: this is a dumb example because "where role != 'admin'" is better
    // but every now and then you need to exclude stuff because of calculations
    // that are a pain in sql and this is a contrived README
    if columns[3] == "admin" {
      return false, []string{}
    }

    extra_column_one = generateSomethingHypotheticalFromColumn(columns[2])
    extra_column_two = lookupSomeApiThingForColumn(columns[4])

    return append(columns, extra_column_one, lookupSomeApiThingForColumn)
})

csvConverter.WriteFile("~/important_user_report.csv")
```

## License

&copy; [John Barton](http://whoisjohnbarton.com/) but under MIT (see [LICENSE](LICENSE)) except for fakedb_test.go which I lifted from the Go standard library and is under BSD and I am unsure what that means legally.
