# Description

[pg2go] is a [PostgreSQL] script that generates [Go] struct definitions for all
tables in a database.

Run it on the database using [psql] and redirect the output to a new Go source
file. Here is a shell session demonstrating this:

```
DB=blogdb
OUT="types_$DB.go"

echo "package main" >"$OUT"
psql -q -t -A -d "$DB" -f pg2go.sql >>"$OUT"
goimports -w "$OUT" || gofmt -w "$OUT"
```

Here we use `head` to peek at the resultant file:

```
head -n 22 types_blogdb.go
package main

import (
    "database/sql"
    "time"
)

type author struct {
    ID         int       `db:"id" json:"id"`
    Created    time.Time `db:"created" json:"created"`
    Name       string    `db:"name" json:"name"`
    Admin      bool      `db:"admin" json:"admin"`
    LoginEmail string    `db:"login_email" json:"login_email"`
    LoginSalt  []byte    `db:"login_salt" json:"login_salt"`
    LoginKey   []byte    `db:"login_key" json:"login_key"`
}

type comment struct {
    ID      int       `db:"id" json:"id"`
    Created time.Time `db:"created" json:"created"`
    Post    int       `db:"post" json:"post"`
    Author  int       `db:"author" json:"author"`
```

# Notes

Using [goimports] rather than standard `gofmt` to format the resultant file has
the benefit of automatically importing packages iff they are required by what
was generated (e.g. `"time"` for `time.Time` & `"database/sql"` for
`sql.NullString`).

Struct fields are tagged `db:"..."` for [package sqlx][sqlx] to pick up on,
should you wish to use it. Similarly, `json:"..."` for `encoding/json`.

A crude [attempt](https://github.com/frou/pg2go/blob/master/pg2go.sql#L78) is
made to singularize plural table names.

If `NEED_GO_TYPE_FOR_...` shows up in the resultant file then add a case for
that type name to the `type_pg2go` function in the `.sql` file.

If the tables you're interested in aren't in the `'public'` schema then search
and replace that in the `.sql` file.

If you want the struct identifiers, and not just their fields, to be exported
(start with upper case) then search and replace `false) AS identifier` with
`true) AS identifier` in the `.sql` file.

# License

```
The MIT License (MIT)

Copyright (c) 2015 Duncan Holm

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

[pg2go]: https://github.com/frou/pg2go
[postgresql]: https://www.postgresql.org
[psql]: http://www.postgresql.org/docs/current/static/app-psql.html
[goimports]: https://godoc.org/golang.org/x/tools/cmd/goimports
[go]: https://www.golang.org
[sqlx]: https://github.com/jmoiron/sqlx
