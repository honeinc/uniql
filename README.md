UniQL (UNIversal Query Language)
=======

Lets you write a simple query and run it against any datastore, like [ElasticSearch](https://github.com/honeinc/uniql-es)
and/or [MongoDB](https://github.com/honeinc/uniql-mongodb).

UniQL parses a simple query langauge into an Abstract Syntax Tree that can then be used to compile a query for a given
datastore. This is useful if you want a level of abstraction between your queries and how your data is actually
stored/searched. You can also use UniQL to execute the same query against multiple datastores at the same time.

For example:

```javascript
var parse = require( 'uniql' );
var mongoCompile = require( 'uniql-mongodb' );
var esCompile = require( 'uniql-es' );

// parse a uniql query into an AST
var queryAST = parse( '( height <= 20 or ( favorites.color == "green" and height != 25 ) ) and firstname ~= "o.+"' );

// using that AST, compile a mongodb query
var mongoQuery = mongoCompile( queryAST );
console.log( util.inspect( mongoQuery, { depth: null } ) );

// using the same AST, compile an elasticsearch query
var esQuery = esCompile( queryAST );
console.log( util.inspect( esQuery, { depth: null } ) );
```

For MongoDB, the query generated is:

```javascript
{ '$or':
   [ { height: { '$lte': 20 } },
     { 'favorites.color': 'green', height: { '$ne': 25 } } ],
  firstname: { '$regex': 'o.+' } }
```

For ElasticSearch, the same query is:

```javascript
{ query:
   { filtered:
      { filter:
         [ { bool:
              { must:
                 [ { bool:
                      { should:
                         [ { range: { height: { lte: 20 } } },
                           { bool:
                              { must:
                                 [ { term: { 'favorites.color': 'green' } },
                                   { bool: { must_not: { term: { height: 25 } } } } ] } } ] } },
                   { bool: { must: { regexp: { firstname: 'o.+' } } } } ] } } ] } } }
```

All generated from one simple query:

````javascript
( height <= 20 or ( favorites.color == "green" and height != 25 ) ) and firstname ~= "o.+"
````

Which produces the following AST:

```javascript
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
- [JavaScript](https://github.com/honeinc/uniql-js)
- !! Your compiler here! Submissions welcome! !!

# UniQL Query Syntax

| Values          | Description                                                               |
| --------------- | ------------------------------------------------------------------------- |
| 43, -1.234      | Numbers                                                                   |
| true, false     | Booleans                                                                  |
| null, undefined | Primitives                                                                |
| "hello"         | Strings                                                                   |
| foo, a.b.c      | Symbols (usually a key or column name in your datastore)                  |

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

# Help and Feedback Wanted

This is an early pass at this. We're very open to getting pull requests to help us improve.

Things we'd love to get to, but would also welcome PRs for:

- Maybe specifying the grammar using [PEG](http://pegjs.org/) would be cleaner/clearer
- Improvements/expansions to the query syntax
- Certain edge case support for things like unary not in the MongoDB driver

# Credits

This was inspired by [FiltrES](https://github.com/abeisgreat/filtres) which was in turn inspired by [Filtrex](https://github.com/joewalnes/filtrex)
