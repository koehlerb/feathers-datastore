# feathers-datastore

[![Build Status](https://travis-ci.org/bedeoverend/feathers-datastore.png?branch=master)](https://travis-ci.org/bedeoverend/feathers-datastore)

> Feathers service for Google Datastore

_Note: this service is still in its early stages and may be buggy and change quickly. Feedback is welcome!_

## Installation

```
npm install @koehlerb/feathers-datastore --save
```

## Documentation

This service follows the standard interface followed by Feathers database services, barring certain caveats as outlined below. Please refer to the [feathers-datastore documentation](http://docs.feathersjs.com/) for Feathers service interface.

### Caveats

(This fork has basic support for $limit and $sort.)

Proper pagination, sorting and certain query filters are not yet implemented. You cannot use: `$in, $nin, $ne, $or`. Some of these will be implemented in future, but may not be due to the limitations of [Google Datastore querying](https://cloud.google.com/datastore/docs/concepts/queries).

It's also worth noting that unless you index your datastore properly, you may run into issues running many combined filters. See [querying Google Datastore](https://cloud.google.com/datastore/docs/concepts/queries).

There is also a performance / cost concern with `$select` - as Google Datastore [cannot project](https://cloud.google.com/datastore/docs/concepts/queries#restrictions_on_queries) unindexed properties, this library fetches the entire entity from the database and then filters it out before sending the response.

### Credentials
If you're wishing to manually authenticate using credentials, pass a [credentials object](https://googlecloudplatform.github.io/google-cloud-node/#/docs/datastore/1.0.0/guides/authentication) to the service factory function e.g.

```js
let credentials = {
  client_email: process.env.GCLOUD_CLIENT_EMAIL,
  private_key: process.env.GCLOUD_PRIVATE_KEY
};

app.use('/user', datastore({ credentials }))
```

_Note: as with Google Cloud, this isn't needed if you're running in an environment which authenticates for you e.g. App Engine or a local machine with the gcloud SDK_

### Kinds
Each service instance should be constructed with a kind for Datastore to use e.g.
```js
app.use('/user', datastore({ kind: 'User' }));
```

This sets `this.kind` to `User`.

### Namespacing
To namespace any operation on the datastore, add it to the query params e.g.

```js
app.service('/user')
  .find({
    query: {
      namespace: 'Marketing'
    }
  });
```

### Datastore Keys
The datastore service has a `makeKey` method which is called to convert a resource `id` into a Datastore Entity key. By default it makes a Key with path `[ this.kind, id ]` and namespace as per the namespace query property.

This method can be overridden in order to serialize / deserialize keys from the ID.

### Finding descendants

To specify an ancestor in a query, add it to the query params e.g.
```js
app.service('/user')
  .find({
    query: {
      ancestor: 'John'
    }
  })
  .then((users) => {
    ...
  });
```

### Indexing
Datastore will error if entity properties are > 1500 bytes and are set to index, so it's worth defining indexing if you can.

Properties can be excluded from indexing by setting the query param `dontIndex` e.g.
```js
app.service('/user')
  .create({
    name: 'Stacey'
    age: 24,
    description: '...'
  }, {
    query: {
      dontIndex: [ 'description' ]
    }
  });
```
`feathers-datastore` can also automatically decide on whether or not to index based on the size of the entity. To enable, set `autoIndex` to true e.g.
```js
app.service('/user')
  .create({
    name: 'Stacey'
    age: 24,
    description: 'Some massive description...'
  }, {
    query: {
      autoIndex: true
    }
  });
```
in which case if for example `description` is larger than 1500 bytes, it won't be indexed.

### Complete Example

Here's an example of a Feathers server that uses `feathers-datastore`.

```js
const feathers = require('feathers');
const rest = require('feathers-rest');
const hooks = require('feathers-hooks');
const bodyParser = require('body-parser');
const errorHandler = require('feathers-errors/handler');
const datastore = require('feathers-datastore');

const kind = 'User';
const projectId = process.env.GCLOUD_PROJECT;

// Initialize the application
const app = feathers()
  .configure(rest())
  .configure(hooks())
  // Needed for parsing bodies (login)
  .use(bodyParser.json())
  .use(bodyParser.urlencoded({ extended: true }))
  // Initialize your feathers datastore
  .use('/user', datastore({ kind, projectId }))
  .use(errorHandler());

app.listen(3030);

console.log('Feathers app started on 127.0.0.1:3030');
```

## License

Copyright (c) 2016

Licensed under the [MIT license](LICENSE).
