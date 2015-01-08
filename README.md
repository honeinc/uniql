UniQL (UNIversal Query Language)
=======

Parses a simple query langauge into an AST that can then be used to compile a query for various datastores.

For example:

````
( height <= 20 or ( favorites.color == "green" and height != 25 ) ) and firstname ~= "o.+"
````

Produces the following AST:

```
{ type: '&&',
  arguments: 
   [ { type: 'EXPRESSION',
       arguments: 
        [ { type: '||',
            arguments: 
             [ { type: '<=',
                 arguments: 
                  [ { type: 'SYMBOL', arguments: [ 'height' ] },
                    { type: 'NUMBER', arguments: [ '20' ] } ] },
               { type: 'EXPRESSION',
                 arguments: 
                  [ { type: '&&',
                      arguments: 
                       [ { type: '==',
                           arguments: 
                            [ { type: 'SYMBOL', arguments: [ 'favorites.color' ] },
                              { type: 'STRING', arguments: [ 'green' ] } ] },
                         { type: '!=',
                           arguments: 
                            [ { type: 'SYMBOL', arguments: [ 'height' ] },
                              { type: 'NUMBER', arguments: [ '25' ] } ] } ] } ] } ] } ] },
     { type: 'MATCH',
       arguments: 
        [ { type: 'SYMBOL', arguments: [ 'firstname' ] },
          { type: 'STRING', arguments: [ 'o.+' ] } ] } ] }
```

Using that AST, you can generate queries for various datastores.

# Available Compilers

- [MongoDB](https://github.com/honeinc/uniql-mongodb)
- [ElasticSearch](https://github.com/honeinc/uniql-es)

# Syntax

| Values     | Description                                                               |
| ---------- | ------------------------------------------------------------------------- |
| 43, -1.234 | Numbers                                                                   |
| "hello"    | Strings                                                                   |
| foo, a.b.c | Symbols (usually a key or column name in your datastore)                  |

| Operators   | Description                                                              |
| ----------- | ------------------------------------------------------------------------ |
| x == y      | Equality                                                                 |
| x != y      | Ineqaulity                                                               |
| x ~= "y"    | Matching evaluated as a RegExp                                           |
| x < y       | Less than                                                                |
| x <= y      | Less than or equal to                                                    |
| x > y       | Greater than                                                             |
| x >= y      | Greater than or equal to                                                 |
| x or y      | Boolean or                                                               |
| x and y     | Boolean and                                                              |
| not x       | Boolean not                                                              |
| ( x )       | Expression                                                               |

Operator precedence follows that of any sane language.

# Credits

This was inspired by [FiltrES](https://github.com/abeisgreat/filtres) which was in turn inspired by [Filtrex](https://github.com/joewalnes/filtrex)
