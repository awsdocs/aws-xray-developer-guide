# Tracing SQL Queries with the X\-Ray SDK for Go<a name="xray-sdk-go-sqlclients"></a>

To trace SQL calls to PostgreSQL or MySQL, replacing `sql.Open` calls to `xray.SQL`, as shown in the following example\. Use URLs instead of configuration strings if possible\.

**Example main\.go**  

```
func main() {
  db := xray.SQL("postgres", "postgres://user:password@host:port/db")
  row, _ := db.QueryRow("SELECT 1") // Use as normal
}
```