# Array Type Mapping

PostgreSQL has the unique feature of supporting [*array data types*](https://www.postgresql.org/docs/current/static/arrays.html). This allow you to conveniently and efficiently store several values in a single column, where in other database you'd typically resort to concatenating the values in a string or defining another table with a one-to-many relationship.

> [!NOTE]
> Although PostgreSQL supports multidimensional arrays, these aren't yet supported by the EF Core provider.

# Mapping arrays

Simply define a regular .NET array or `List<>` property, and the provider

```c#
public class Post
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string[] Tags { get; set; }
    public List<string> AlternativeTags { get; set; }
}
```

The provider will create `text[]` columns for the above two properties, and will properly detect changes in them - if you load an array and change one of its elements, calling `SaveChanges()` will automatically update the row in the database accordingly.

# Operation translation

The provider can also translate CLR array operations to the corresponding SQL operation; this allows you to efficiently work with arrays by evaluating operations in the database and avoids pulling all the data. The following table lists the range operations that currently get translated. If you run into a missing operation, please open an issue.

Note that operation translation on `List<>` is limited at this time, but will be improved in the future. It's recommended to use an array for now.

| C# expression                                              | SQL generated by Npgsql |
|------------------------------------------------------------|-------------------------|
| `.Where(c => c.SomeArray[1] == "foo")`                     | [`WHERE "c"."SomeArray"[1] = 'foo'`](https://www.postgresql.org/docs/current/static/arrays.html#ARRAYS-ACCESSING)
| `.Where(c => c.SomeArray.SequenceEqual(new[] { 1, 2, 3 })` | [`WHERE "c"."SomeArray" = ARRAY[1, 2, 3])`](https://www.postgresql.org/docs/current/static/arrays.html)
| `.Where(c => c.SomeArray.Contains(3))`                     | [`WHERE 3 = ANY("c"."SomeArray")`](https://www.postgresql.org/docs/current/static/functions-comparisons.html#AEN21104)
| `.Where(c => c.SomeArray.Length == 3)`                     | [`WHERE cardinality("c"."SomeArray") = 3`](https://www.postgresql.org/docs/current/static/functions-array.html#ARRAY-FUNCTIONS-TABLE)