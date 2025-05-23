# Other Methods

## debug()

Chaining this method to any query will print all the executed SQL to console.

See [knex documentation](https://knexjs.org/guide/query-builder.html#debug)

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## toKnexQuery()

```js
knexQueryBuilder = queryBuilder.toKnexQuery();
```

Compiles the query into a knex query and returns the knex query builder instance.

Some objection queries, like when `withGraphFetched` is used, actually execute multiple queries. In these cases, the query builer of the first query is returned

In some rare cases the knex query cannot be built synchronously. In these cases you will get a clear error message. These cases can be handled by calling the optional [initialize](/api/objection/#initialize) method once before the failing `toKnexQuery` method is called.

```js
const { initialize } = require('objection');

await initialize([Person, Pet, Movie, SomeOtherModelClass]);
```

## for()

```js
queryBuilder = queryBuilder.for(relationOwner);
```

This method can only be used in conjunction with the static [relatedQuery](/api/model/static-methods.html#static-relatedquery) method. See the [relatedQuery](/api/model/static-methods.html#static-relatedquery) documentation on how to use it.

This method takes one argument (the owner(s) of the relation) and it can have any of the following types:

- A single identifier (can be composite)
- An array of identifiers (can be composite)
- A [QueryBuilder](/api/query-builder/)
- A model instance.
- An array of model instances.

## context()

```js
queryBuilder = queryBuilder.context(queryContext);
```

Sets/gets the query context.

Some query builder methods create more than one query. The query context is an object that is shared with all queries started by a query builder.

The context is also passed to [\$beforeInsert](/api/model/instance-methods.html#beforeinsert), [\$afterInsert](/api/model/instance-methods.html#afterinsert), [\$beforeUpdate](/api/model/instance-methods.html#beforeupdate), [\$afterUpdate](/api/model/instance-methods.html#afterupdate), [\$beforeDelete](/api/model/instance-methods.html#beforedelete), [\$afterDelete](/api/model/instance-methods.html#afterdelete) and [\$afterFind](/api/model/instance-methods.html#afterfind) calls that the query creates.

In addition to properties added using this method the query context object always has a `transaction` property that holds the active transaction. If there is no active transaction the `transaction` property contains the normal knex instance. In both cases the value can be passed anywhere where a transaction object can be passed so you never need to check for the existence of the `transaction` property.

This method merges the given object with the current context. You can use `clearContext` to clear the context if needed.

See the methods [runBefore](/api/query-builder/other-methods.html#runbefore), [onBuild](/api/query-builder/other-methods.html#onbuild) and [runAfter](/api/query-builder/other-methods.html#runafter)
for more information about the hooks.

##### Arguments

| Argument     | Type   | Description              |
| ------------ | ------ | ------------------------ |
| queryContext | Object | The query context object |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

You can set the context like this:

```js
await Person.query().context({ something: 'hello' });
```

and access the context like this:

```js
const context = builder.context();
```

You can set any data to the context object. You can also register QueryBuilder lifecycle methods for _all_ queries that share the context:

```js
Person.query().context({
  runBefore(result, builder) {
    return result;
  },
  runAfter(result, builder) {
    return result;
  },
  onBuild(builder) {}
});
```

For example the `withGraphFetched` method causes multiple queries to be executed from a single query builder. If you wanted to make all of them use the same schema you could write this:

```js
Person.query()
  .withGraphFetched('[movies, children.movies]')
  .context({
    onBuild(builder) {
      builder.withSchema('someSchema');
    }
  });
```

## clearContext()

```js
queryBuilder = queryBuilder.clearContext();
```

Replaces the current context with an empty object.

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## tableNameFor()

```js
const tableName = queryBuilder.tableNameFor(modelClass);
```

Returns the table name for a given model class in the query. Usually the table name can be fetched through `Model.tableName` but if the source table has been changed for example using the [QueryBuilder#table](/api/query-builder/find-methods.html#table) method `tableNameFor` will return the correct value.

##### Arguments

| Argument   | Type     | Description    |
| ---------- | -------- | -------------- |
| modelClass | function | A model class. |

##### Return value

| Type   | Description                                       |
| ------ | ------------------------------------------------- |
| string | The source table (or view) name for `modelClass`. |

## tableRefFor()

```js
const tableRef = queryBuilder.tableRefFor(modelClass);
```

Returns the name that should be used to refer to the `modelClass`'s table in the query.
Usually a table can be referred to using its name, but `tableRefFor` can return a different
value for example in case an alias has been given.

##### Arguments

| Argument   | Type     | Description    |
| ---------- | -------- | -------------- |
| modelClass | function | A model class. |

##### Return value

| Type   | Description                                                    |
| ------ | -------------------------------------------------------------- |
| string | The name that should be used to refer to a table in the query. |

## reject()

```js
queryBuilder = queryBuilder.reject(reason);
```

Skips the database query and "fakes" an error result.

##### Arguments

| Argument | Type | Description          |
| -------- | ---- | -------------------- |
| reason   |      | The rejection reason |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## resolve()

```js
queryBuilder = queryBuilder.resolve(value);
```

Skips the database query and "fakes" a result.

##### Arguments

| Argument | Type | Description       |
| -------- | ---- | ----------------- |
| value    |      | The resolve value |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## isExecutable()

```js
const isExecutable = queryBuilder.isExecutable();
```

Returns `false` if this query will never be executed.

This may be true in multiple cases:

1. The query is explicitly resolved or rejected using the [resolve](/api/query-builder/other-methods.html#resolve) or [reject](/api/query-builder/other-methods.html#reject) methods.
2. The query starts a different query when it is executed.

##### Return value

| Type    | Description                                |
| ------- | ------------------------------------------ |
| boolean | `false` if the query will never be executed. |

## isFind()

```js
const isFind = queryBuilder.isFind();
```

Returns `true` if the query is read-only.

##### Return value

| Type    | Description                     |
| ------- | ------------------------------- |
| boolean | `true` if the query is read-only. |

## isInsert()

```js
const isInsert = queryBuilder.isInsert();
```

Returns `true` if the query performs an insert operation.

##### Return value

| Type    | Description                                     |
| ------- | ----------------------------------------------- |
| boolean | `true` if the query performs an insert operation. |

## isUpdate()

```js
const isUpdate = queryBuilder.isUpdate();
```

Returns `true` if the query performs an update or patch operation.

##### Return value

| Type    | Description                                              |
| ------- | -------------------------------------------------------- |
| boolean | `true` if the query performs an update or patch operation. |

## isDelete()

```js
const isDelete = queryBuilder.isDelete();
```

Returns `true` if the query performs a delete operation.

##### Return value

| Type    | Description                                    |
| ------- | ---------------------------------------------- |
| boolean | `true` if the query performs a delete operation. |

## isRelate()

```js
const isRelate = queryBuilder.isRelate();
```

Returns `true` if the query performs a relate operation.

##### Return value

| Type    | Description                                    |
| ------- | ---------------------------------------------- |
| boolean | `true` if the query performs a relate operation. |

## isUnrelate()

```js
const isUnrelate = queryBuilder.isUnrelate();
```

Returns `true` if the query performs an unrelate operation.

##### Return value

| Type    | Description                                       |
| ------- | ------------------------------------------------- |
| boolean | `true` if the query performs an unrelate operation. |

## isInternal()

```js
const isInternal = queryBuilder.isInternal();
```

Returns `true` for internal "helper" queries that are not directly
part of the operation being executed. For example the `select` queries
performed by `upsertGraph` to get the current state of the graph are
internal queries.

##### Return value

| Type    | Description                                              |
| ------- | -------------------------------------------------------- |
| boolean | `true` if the query performs an internal helper operation. |

## hasWheres()

```js
const hasWheres = queryBuilder.hasWheres();
```

Returns `true` if the query contains where statements.

##### Return value

| Type    | Description                                  |
| ------- | -------------------------------------------- |
| boolean | `true` if the query contains where statements. |

## hasSelects()

```js
const hasSelects = queryBuilder.hasSelects();
```

Returns `true` if the query contains any specific select staments, such as:
`'select'`, `'columns'`, `'column'`, `'distinct'`, `'count'`, `'countDistinct'`, `'min'`, `'max'`, `'sum'`, `'sumDistinct'`, `'avg'`, `'avgDistinct'`

##### Return value

| Type    | Description                                              |
| ------- | -------------------------------------------------------- |
| boolean | `true` if the query contains any specific select staments. |

## hasWithGraph()

```js
const hasWithGraph = queryBuilder.hasWithGraph();
```

Returns `true` if `withGraphFetched` or `withGraphJoined` has been called for the query.

##### Return value

| Type    | Description                                                                    |
| ------- | ------------------------------------------------------------------------------ |
| boolean | `true` if `withGraphFetched` or `withGraphJoined` has been called for the query. |

## has()

```js
const has = queryBuilder.has(selector);
```

```js
console.log(
  Person.query()
    .range(0, 4)
    .has('range')
);
```

Returns `true` if the query defines an operation that matches the given selector.

##### Arguments

| Argument | Type                           | Description                                                           |
| -------- | ------------------------------ | --------------------------------------------------------------------- |
| selector | string&nbsp;&#124;&nbsp;RegExp | A name or regular expression to match all defined operations against. |

##### Return value

| Type    | Description                                                             |
| ------- | ----------------------------------------------------------------------- |
| boolean | `true` if the query defines an operation that matches the given selector. |

## clear()

```js
queryBuilder = queryBuilder.clear(selector);
```

Removes all operations in the query that match the given selector.

##### Arguments

| Argument | Type                           | Description                                                                          |
| -------- | ------------------------------ | ------------------------------------------------------------------------------------ |
| selector | string&nbsp;&#124;&nbsp;regexp | A name or regular expression to match all operations that are to be removed against. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
console.log(
  Person.query()
    .orderBy('firstName')
    .clear('orderBy')
    .has('orderBy')
);
```

## runBefore()

```js
queryBuilder = queryBuilder.runBefore(runBefore);
```

Registers a function to be called before just the database query when the builder is executed. Multiple functions can be chained like `then` methods of a promise.

##### Arguments

| Argument  | Type                                                       | Description                                                                                                                                         |
| --------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| runBefore | function(result,&nbsp;[QueryBuilder](/api/query-builder/)) | The function to be executed. This function can be async. Note that it needs to return the result used for further processing in the chain of calls. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
const query = Person.query();

query
  .runBefore(async result => {
    console.log('hello 1');

    await Promise.delay(10);

    console.log('hello 2');
    return result;
  })
  .runBefore(result => {
    console.log('hello 3');
    return result;
  });

await query;
// --> hello 1
// --> hello 2
// --> hello 3
```

## onBuild()

```js
queryBuilder = queryBuilder.onBuild(onBuild);
```

Functions registered with this method are called each time the query is built into an SQL string. This method is ran after [runBefore](/api/query-builder/other-methods.html#runbefore) methods but before [runAfter](/api/query-builder/other-methods.html#runafter) methods.

If you need to modify the SQL query at query build time, this is the place to do it. You shouldn't modify the query in any of the `run` methods.

Unlike the `run` methods (`runAfter`, `runBefore` etc.) these must be synchronous. Also you should not register any `run` methods from these. You should _only_ call the query building methods of the builder provided as a parameter.

##### Arguments

| Argument | Type                                          | Description                                  |
| -------- | --------------------------------------------- | -------------------------------------------- |
| onBuild  | function([QueryBuilder](/api/query-builder/)) | The **synchronous** function to be executed. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Eamples

```js
const query = Person.query();

query
  .onBuild(builder => {
    builder.where('id', 1);
  })
  .onBuild(builder => {
    builder.orWhere('id', 2);
  });
```

## onBuildKnex()

```js
queryBuilder = queryBuilder.onBuildKnex(onBuildKnex);
```

Functions registered with this method are called each time the query is built into an SQL string. This method is ran after [onBuild](/api/query-builder/other-methods.html#onbuild) methods but before [runAfter](/api/query-builder/other-methods.html#runafter) methods.

If you need to modify the SQL query at query build time, this is the place to do it in addition to `onBuild`. The only difference between `onBuildKnex` and `onBuild` is that in `onBuild` you can modify the objection's query builder. In `onBuildKnex` the objection builder has been compiled into a knex query builder and any modifications to the objection builder will be ignored.

Unlike the `run` methods (`runAfter`, `runBefore` etc.) these must be synchronous. Also you should not register any `run` methods from these. You should _only_ call the query building methods of the **knexBuilder** provided as a parameter.

::: warning
You should never call any query building (or any other mutating) method on the `objectionBuilder` in this function. If you do, those calls will get ignored. At this point the query builder has been compiled into a knex query builder and you should only modify that. You can call non mutating methods like `hasSelects`, `hasWheres` etc. on the objection builder.
:::

##### Arguments

| Argument    | Type                                                                   | Description                  |
| ----------- | ---------------------------------------------------------------------- | ---------------------------- |
| onBuildKnex | function(`KnexQueryBuilder`,&nbsp;[QueryBuilder](/api/query-builder/)) | The function to be executed. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
const query = Person.query();

query.onBuildKnex((knexBuilder, objectionBuilder) => {
  knexBuilder.where('id', 1);
});
```

## runAfter()

```js
queryBuilder = queryBuilder.runAfter(runAfter);
```

Registers a function to be called when the builder is executed.

These functions are executed as the last thing before any promise handlers registered using the [then](/api/query-builder/other-methods.html#then) method. Multiple functions can be chained like [then](/api/query-builder/other-methods.html#then) methods of a promise.

##### Arguments

| Argument | Type                                                       | Description                                                                                                                                         |
| -------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| runAfter | function(result,&nbsp;[QueryBuilder](/api/query-builder/)) | The function to be executed. This function can be async. Note that it needs to return the result used for further processing in the chain of calls. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
const query = Person.query();

query
  .runAfter(async (models, queryBuilder) => {
    return models;
  })
  .runAfter(async (models, queryBuilder) => {
    models.push(Person.fromJson({ firstName: 'Jennifer' }));
    return models;
  });

const models = await query;
```

## onError()

```js
queryBuilder = queryBuilder.onError(onError);
```

Registers an error handler. Just like `catch` but doesn't execute the query.

##### Arguments

| Argument | Type                                                      | Description                           |
| -------- | --------------------------------------------------------- | ------------------------------------- |
| onError  | function(Error,&nbsp;[QueryBuilder](/api/query-builder/)) | The function to be executed on error. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
const query = Person.query();

query
  .onError(async (error, queryBuilder) => {
    // Handle `SomeError` but let other errors go through.
    if (error instanceof SomeError) {
      // This will cause the query to be resolved with an object
      // instead of throwing an error.
      return { error: 'some error occurred' };
    } else {
      return Promise.reject(error);
    }
  })
  .where('age', '>', 30);
```

## castTo()

```js
queryBuilder = queryBuilder.castTo(ModelClass);
```

```ts
queryBuilder = queryBuilder.castTo<SomeType>();
```

Sets the model class of the result rows or if no arguments are provided, simply casts the
result type to the provided type.

##### Return value

| Type                      | Description                         |
| ------------------------- | ----------------------------------- |
| [ModelClass](/api/model/) | The model class of the result rows. |

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

The following example creates a query through `Person`, joins a bunch of relations, selects
only the related `Animal`'s columns and returns the results as `Animal` instances instead
of `Person` instances.

```js
const animals = await Person.query()
  .joinRelated('children.children.pets')
  .select('children:children:pets.*')
  .castTo(Animal);
```

If you don't provide any arguments for the method, but provide a generic type argument, the
result is not changed but its typescript type is cast to the given generic type.

```ts
interface Named {
  name: string;
}

const result = await Person.query()
  .select('firstName as name')
  .castTo<Named[]>();

console.log(result[0].name);
```

## modelClass()

```js
const modelClass = queryBuilder.modelClass();
```

Gets the Model subclass this builder is bound to.

##### Return value

| Type                 | Description                                 |
| -------------------- | ------------------------------------------- |
| [Model](/api/model/) | The Model subclass this builder is bound to |

## skipUndefined()

```js
queryBuilder = queryBuilder.skipUndefined();
```

If this method is called for a builder then undefined values passed to the query builder methods don't cause an exception but are ignored instead.

For example the following query will return all `Person` rows if `req.query.firstName` is `undefined`.

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

##### Examples

```js
Person.query()
  .skipUndefined()
  .where('firstName', req.query.firstName);
```

## transacting()

```js
queryBuilder = queryBuilder.transacting(transaction);
```

Sets the transaction for a query.

##### Arguments

| Argument    | Type   | Description          |
| ----------- | ------ | -------------------- |
| transaction | object | A transaction object |

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

## clone()

```js
const clone = queryBuilder.clone();
```

Create a clone of this builder.

| Type                                | Description                |
| ----------------------------------- | -------------------------- |
| [QueryBuilder](/api/query-builder/) | Clone of the query builder |

## execute()

```js
const promise = queryBuilder.execute();
```

Executes the query and returns a Promise.

##### Return value

| Type      | Description                                                |
| --------- | ---------------------------------------------------------- |
| `Promise` | Promise the will be resolved with the result of the query. |

## then()

```js
const promise = queryBuilder.then(successHandler, errorHandler);
```

Executes the query and returns a Promise.

##### Arguments

| Argument       | Type     | Default  | Description             |
| -------------- | -------- | -------- | ----------------------- |
| successHandler | function | identity | Promise success handler |
| errorHandler   | function | identity | Promise error handler   |

##### Return value

| Type      | Description                                                |
| --------- | ---------------------------------------------------------- |
| `Promise` | Promise the will be resolved with the result of the query. |

## catch()

```js
const promise = queryBuilder.catch(errorHandler);
```

Executes the query and calls `catch(errorHandler)` for the returned promise.

##### Arguments

| Argument     | Type     | Default  | Description   |
| ------------ | -------- | -------- | ------------- |
| errorHandler | function | identity | Error handler |

##### Return value

| Type      | Description                                                |
| --------- | ---------------------------------------------------------- |
| `Promise` | Promise the will be resolved with the result of the query. |

## bind()

```js
const promise = queryBuilder.bind(returnValue);
```

Executes the query and calls `bind(context)` for the returned promise.

##### Arguments

| Argument | Type | Default   | Description  |
| -------- | ---- | --------- | ------------ |
| context  |      | undefined | Bind context |

##### Return value

| Type      | Description                                                |
| --------- | ---------------------------------------------------------- |
| `Promise` | Promise the will be resolved with the result of the query. |

## resultSize()

```js
const promise = queryBuilder.resultSize();
```

Returns the amount of rows the current query would produce without [limit](/api/query-builder/find-methods.html#limit) and [offset](/api/query-builder/find-methods.html#offset) applied. Note that this executes a copy of the query and returns a Promise.

This method is often more convenient than `count` which returns an array of objects instead a single number.

##### Return value

| Type              | Description                                        |
| ----------------- | -------------------------------------------------- |
| `Promise<number>` | Promise the will be resolved with the result size. |

##### Examples

```js
const query = Person.query().where('age', '>', 20);

const [total, models] = await Promise.all([
  query.resultSize(),
  query.offset(100).limit(50)
]);
```

## page()

```js
queryBuilder = queryBuilder.page(page, pageSize);
```

```js
const result = await Person.query()
  .where('age', '>', 20)
  .page(5, 100);

console.log(result.results.length); // --> 100
console.log(result.total); // --> 3341
```

Two queries are performed by this method: the actual query and a query to get the `total` count.

Mysql has the `SQL_CALC_FOUND_ROWS` option and `FOUND_ROWS()` function that can be used to calculate the result size, but according to my tests and [the interwebs](https://www.google.com/search?q=SQL_CALC_FOUND_ROWS+performance) the performance is significantly worse than just executing a separate count query.

Postgresql has window functions that can be used to get the total count like this `select count(*) over () as total`. The problem with this is that if the result set is empty, we don't get the total count either. (If someone can figure out a way around this, a PR is very welcome).

##### Arguments

| Argument | Type   | Description                                                        |
| -------- | ------ | ------------------------------------------------------------------ |
| page     | number | The index of the page to return. The index of the first page is 0. |
| pageSize | number | The page size                                                      |

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

## range()

```js
queryBuilder = queryBuilder.range(start, end);
```

Only returns the given range of results.

Two queries are performed by this method: the actual query and a query to get the `total` count.

Mysql has the `SQL_CALC_FOUND_ROWS` option and `FOUND_ROWS()` function that can be used to calculate the result size, but according to my tests and [the interwebs](https://www.google.com/search?q=SQL_CALC_FOUND_ROWS+performance) the performance is significantly worse than just executing a separate count query.

Postgresql has window functions that can be used to get the total count like this `select count(*) over () as total`. The problem with this is that if the result set is empty, we don't get the total count either. (If someone can figure out a way around this, a PR is very welcome).

##### Arguments

| Argument | Type   | Description                               |
| -------- | ------ | ----------------------------------------- |
| start    | number | The index of the first result (inclusive) |
| end      | number | The index of the last result (inclusive)  |

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

##### Examples

```js
const result = await Person.query()
  .where('age', '>', 20)
  .range(0, 100);

console.log(result.results.length); // --> 101
console.log(result.total); // --> 3341
```

`range` can be called without arguments if you want to specify the limit and offset explicitly:

```js
const result = await Person.query()
  .where('age', '>', 20)
  .limit(10)
  .range();

console.log(result.results.length); // --> 101
console.log(result.total); // --> 3341
```

## first()

```js
queryBuilder = queryBuilder.first();
```

If the result is an array, selects the first item.

NOTE: This doesn't add `limit 1` to the query by default. You can override the [Model.useLimitInFirst](/api/model/static-properties.html#static-uselimitinfirst) property to change this behaviour.

Also see [findById](/api/query-builder/find-methods.html#findbyid) and [findOne](/api/query-builder/find-methods.html#findone) shorthand methods.

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

##### Examples

```js
const firstPerson = await Person.query().first();

console.log(firstPerson.age);
```

## throwIfNotFound()

```js
queryBuilder = queryBuilder.throwIfNotFound(data);
```

Causes a [Model.NotFoundError](/api/types/#class-notfounderror) to be thrown if the query result is empty.

You can replace `Model.NotFoundError` with your own error by implementing the static [Model.createNotFoundError(ctx)](/api/model/static-methods.html#static-createnotfounderror) method.

##### Arguments

| Argument | Type   | Description                                                                                                                                                                                                                                                                                                                      |
| -------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| data     | object | optional object with custom data may contain any property such as message, type. The object is returned in the error thrown under the `data` property. A special case is for the optional message property. This is used to set the title of the error. These extra properties can be leveraged in the error handling middleware |

##### Return value

| Type                                | Description                       |
| ----------------------------------- | --------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining |

##### Examples

```js
try {
  await Language.query()
    .where('name', 'Java')
    .andWhere('isModern', true)
    .throwIfNotFound({
      message: `Custom message returned`,
      type: `Custom type`
    });
} catch (err) {
  // No results found.
  console.log(err instanceof Language.NotFoundError); // --> true
}
```

## timeout()

See [knex documentation](https://knexjs.org/guide/query-builder.html#debug)

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## connection()

See [knex documentation](https://knexjs.org/guide/query-builder.html#connection)

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## modify()

Works like `knex`'s [modify](https://knexjs.org/guide/query-builder.html#modify) function but in addition you can specify a [modifier](/api/model/static-properties.html#static-modifiers) by providing modifier names.

See the [modifier](/recipes/modifiers.html) recipe for examples of the things you can do with modifiers.

##### Arguments

| Argument    | Type                                                                                            | Description                                                                                                                                                                                                                                                          |
| ----------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| modifier    | function([QueryBuilder](/api/query-builder/))&nbsp;&#124;&nbsp;string&nbsp;&#124;&nbsp;string[] | The modify callback function, receiving the builder as its first argument, followed by the optional arguments. If a string or an array of strings is provided, the corresponding [modifier](/api/model/static-properties.html#static-modifiers) is executed instead. |
| \*arguments | ...any                                                                                          | The optional arguments passed to the modify function                                                                                                                                                                                                                 |

##### Examples

The first argument can be a name of a [model modifier](/api/model/static-properties.html#static-modifiers). The rest of the arguments are passed as arguments for the modifier.

```js
Person.query().modify('someModifier', 'foo', 1);
```

You can also pass an array of modifier names:

```js
Person.query().modify(['someModifier', 'someOtherModifier'], 'foo', 1);
```

The first argument can be a function:

```js
function modifierFunc(query, arg1, arg2) {
  query.where(arg1, arg2);
}

Person.query().modify(modifierFunc, 'foo', 1);
```

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

## modifiers()

Registers modifiers for the query.

You can call this method without arguments to get the currently registered modifiers.

See the [modifier recipe](/recipes/modifiers.html) for more info and examples.

##### Return value

| Type                                | Description                        |
| ----------------------------------- | ---------------------------------- |
| [QueryBuilder](/api/query-builder/) | `this` query builder for chaining. |

##### Examples

```js
const people = await Person.query()
  .modifiers({
    selectFields: query => query.select('id', 'name'),
    // In the following modifier, `filterGender` is a modifier
    // registered in Person.modifiers object. Query modifiers
    // can be used to bind arguments to model modifiers like this.
    filterWomen: query => query.modify('filterGender', 'female')
  })
  .modify('selectFields')
  .withGraphFetched('children(selectFields, filterWomen)');
```

You can get the currently registered modifiers by calling the method without arguments.

```js
const modifiers = query.modifiers();
```
