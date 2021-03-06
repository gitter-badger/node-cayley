# node-cayley [![Build Status](https://travis-ci.org/lnshi/node-cayley.svg?branch=master)](https://travis-ci.org/lnshi/node-cayley) [![Coverage Status](https://coveralls.io/repos/github/lnshi/node-cayley/badge.svg?branch=master)](https://coveralls.io/github/lnshi/node-cayley?branch=master) [![npm version](https://badge.fury.io/js/node-cayley.svg)](https://badge.fury.io/js/node-cayley)

This is a Node.js client for open-source graph database [cayley](https://github.com/cayleygraph/cayley).

## Feature list

* ES6 based.
* TLS supported.
* Multiple cayley hosts supported.
* Default random client selection strategy.
* Bluebird Promise style and callback style APIs.
* Transparent bidirectional JSON <-> N-Quads data formatting handling.
* Travis CI.
* Fully covered mocha + chai test cases.
* Clearly designed entry-level N-Quads data: [friend_circle_with_label.nq](./test/data/friend_circle_with_label.nq) and [equivalent JSON](./test/data/friend_circle_with_label.nq.equivalent_json.js) for getting you in.

## Todo list

* [JSON-LD](http://json-ld.org/) support.

## Visualized graph

> **Note:**

> All the code examples in this README guidebook are based on the N-Quads test data here: [friend_circle_with_label.nq](./test/data/friend_circle_with_label.nq), which can be visualized as the below graph in cayley:

```
_:A  standards for the value:  _:BN@</user/shortid/23TplPdS>.<mobilePhone>
_:B  standards for the value:  _:BN@</user/shortid/46Juzcyx>.<mobilePhone>
_:C  standards for the value:  _:BN@</user/shortid/hwX6aOr7>.<mobilePhone>
```

**[⇩ Catalog](#catalog)**

<img src="https://github.com/lnshi/node-cayley/blob/master/test/data/friend_circle_with_label.nq_0_visualized.png" />

**[⇩ Catalog](#catalog)**

# Documentation

## Catalog

* [Visualized graph](#visualized-graph)
* [Basic usages examples](#basic-usages-examples)
* [Configuration](#configuration)
* [Default random client selection strategy](#default-random-client-selection-strategy)
* [HTTP APIs](#http-apis)
  * [write(data, callback)](#writedata-callback)
  * [read(callback)](#readcallback)
  * [delete(data, callback)](#deletedata-callback)
* [Gizmo APIs → graph object](#gizmo-apis--graph-object)
  * [graph object](#graph-object)
  * [graph.type(type)](#graphtypetype)
  * [graph.Vertex([nodeId])](#graphvertexnodeid)
  * [graph.Morphism()](#graphmorphism)
  * [graph.Emit(data)](#graphemitdata)
* [Gizmo APIs → Path Object](#gizmo-apis--path-object)
  * [path.Out([predicatePath], [tag])](#pathoutpredicatepath-tag)
  * [path.In([predicatePath], [tag])](#pathinpredicatepath-tag)
  * [path.Both([predicatePath], [tag])](#pathbothpredicatepath-tag)
  * [path.Is([node])](#pathisnode)
  * [path.Has(predicatePath, node)](#pathhaspredicatepath-node)
  * [path.LabelContext([labelPath], [tag])](#pathlabelcontextlabelpath-tag)
  * [path.Limit(limit)](#pathlimitlimit)
  * [path.Skip(offset)](#pathskipoffset)
  * [path.InPredicates()](#pathinpredicates)
  * [path.OutPredicates()](#pathoutpredicates)
  * [path.Tag([tag])](#pathtagtag)
  * [path.Back(tag)](#pathbacktag)
  * [path.Save(predicate, tag)](#pathsavepredicate-tag)
  * [path.Intersect(query)](#pathintersectquery)
  * [path.Union(query)](#pathunionquery)
  * [path.Except(query)](#pathexceptquery)
  * [path.Follow(morphism)](#pathfollowmorphism)
  * [path.FollowR(morphism)](#pathfollowrmorphism)
* [Gizmo APIs → Query Object](#gizmo-apis--query-object)
  * [query.All(callback)](#queryallcallback)
  * [query.GetLimit(size, callback)](#querygetlimitsize-callback)
  * [query.ToArray(gizmoFunc, callback)](#querytoarraygizmofunc-callback)
  * [query.TagArray(gizmoFunc, callback)](#querytagarraygizmofunc-callback)
  * [query.ToValue(gizmoFunc, callback)](#querytovaluegizmofunc-callback)
  * [query.TagValue(gizmoFunc, callback)](#querytagvaluegizmofunc-callback)
  * [query.ForEach(limit, gizmoFunc, callback)](#queryforeachlimit-gizmofunc-callback)
* [Additional Resources](#additional-resources)

## Basic usages examples

**[⇪ Catalog](#catalog)**

```
npm install node-cayley --save
```

```javascript
const client = require('node-cayley')('localhost:64210');

const g = graph = client.g; // Or: const g = graph = client.graph;

client.write(jsonObjArr).then((res) => {
  // Successfully wrote to cayley.
}).catch((err) => {
  // Error ...
});

// Callback style.
client.delete(jsonObjArr, (err, res) => {});

g.V('</user/shortid/46Juzcyx>').Out('<follows>', 'predicate').All().then((res) => {
  // res will be: {result:[{id:'</user/shortid/23TplPdS>',predicate:'<follows>'}]}
}).catch((err) => {
  // Error ...
});

// 'type' default to 'query', you can change to 'shape' by calling g.type('shape')
g.type('shape').V().All((err, res) => {});

const popularQuery = g.M().Out('<follows>').In('<follows>')
                      .Has('<gender>', 'F').Out(['<email>', '<mobilePhone>']);

g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).All().then((res) => {
  // res will be:
  //   {result:[{id:'_:C'},{id:'xxx.l30@xxx.com'}]}
}).catch((err) => {
  // Error ...
});
```

**[⇪ Catalog](#catalog)**

## Configuration

**[⇪ Catalog](#catalog)**

  * Options

    * promisify: `boolean`, default to true, set true to use bluebird style APIs, false to use callback style APIs.

    * logLevel: `integer`, set log level, this lib use logger winston and npm log levels(winston.config.npm.levels).

    * secure: `boolean`, set whether your cayley server use TLS.

    * certFile: `string`, path of your TLS cert file, if above 'secure' is set to true, this must be provided.

    * keyFile: `string`, path of your TLS key file, if above 'secure' is set to true, this must be provided.

    * caFile: `string`, path of your TLS root CA file, if above 'secure' is set to true, this must be provided.

    * servers: `Array of multiple cayley hosts object`, options here will override the top level options.

  * Two configuration examples:

    ```javascript
    const client = require('node-cayley')('localhost:64210', {
      promisify: true
    });

    const g = graph = client.g;
    ```
    ```javascript
    const clients = require('node-cayley')({
      promisify: true,
      secure: true,
      certFile: '',
      keyFile: '',
      caFile: '',
      servers: [
        {
          host: host_0,
          port: port_0
        }, {
          host: host_1,
          port: port_1,
          secure: false
        }
      ]
    });

    clients.pickRandomly().read().then((res) => {/* Your data in JSON. */}).catch((err) => {});

    const g = graph = clients.pickRandomly().g;

    g.V().All().then((res) => {/* Your data in JSON. */}).catch((err) => {});
    ```

**[⇪ Catalog](#catalog)**

## Default random client selection strategy

**[⇪ Catalog](#catalog)**

  * If single cayley host is provided, the lib will return one single client directly.

  * If multiple cayley hosts are provided, the lib will return a clients array plus a default random client selection strategy which is named as `pickRandomly`, you can just use it as:

    ```javascript
    const cayleyClients = require('node-cayley')({
      servers: [
        {
          host: host_0,
          port: port_0
        },
        {
          host: host_1,
          port: port_1
        }
      ]
    });

    const client = cayleyClients.pickRandomly();

    const g = graph = client.g;

    client.delete(jsonObjArr).then((res) => {/* ... */}).catch((err) => {});
    ```

**[⇪ Catalog](#catalog)**

## HTTP APIs

> Depends on your `promisify` setting, this lib will provide `bluebird Promise style` or `callback style` API, for both styles, usages examples can be found in the lib [test folder](./test).

**[⇪ Catalog](#catalog)**

### write(data, callback)

**[⇪ Catalog](#catalog)**

* Description: write your JSON data into cayley as N-Quads data transparently.

* **data**: Array of JSON objects, you need to add the below two extra fields to each object:

  * **primaryKey**: `required`, which will be the **Subject** in the N-Quads data.

    > Note: you need to define a way to generate consistent `primaryKey` for same data, the exactly same `primaryKey`(Subject) is required when in the future you try to delete this N-Quads entry.

  * **label**: `optional`, which is for cayley subgraph organizing.

* **callback(err, res)**

* Usage example:
  
  ```javascript
  client.write([
    {
      primaryKey: '</user/shortid/23TplPdS>',
      label: 'companyA',

      userId: '23TplPdS',
      realName: 'XXX_L3'
    }
  ], (err, res) => {
    if (err) {
      // Something went wrong...
    } else {
      // resBody: cayley server response body to this write.
    }
  });
  ```

**[⇪ Catalog](#catalog)**

### read(callback)

**[⇪ Catalog](#catalog)**

* Description: read N-Quads data from cayley, the lib will transparently convert the data to JSON.
  > Note: response not in JSON yet, will add the functionality soon.

* **callback(err, res)**

* Usage example:

  ```javascript
  client.read().then((res) => {
    // Your data in JSON.
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### delete(data, callback)

**[⇪ Catalog](#catalog)**

* Description: delete the corresponding N-Quads data which are represented by this JSON data from cayley transparently.

* **data**: Array of JSON objects, for each object you need to provide the values for the below two extra fields:

  * **primaryKey**: `required`, which should be exactly same with the 'primaryKey' when you inserted this data.

  * **label**: if you provided the `label` when you inserted this data, then for deleting you must also need to provide the exactly same value.

* **callback(err, res)**

* Usage example:
  
  ```javascript
  client.delete([
    {
      primaryKey: '</user/shortid/23TplPdS>',
      label: 'companyA',

      userId: '23TplPdS',
      realName: 'XXX_L3'
    }
  ]).then((res) => {
    // Successfully deleted from cayley.
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

## Gizmo APIs → graph object

### graph object

* grpah
  > const g = client.graph;

* Alias: `g`
  > const g = client.g;

* This is the only special object in the environment, generates the query objects. Under the hood, they're simple objects that get compiled to a Go iterator tree when executed.

**[⇪ Catalog](#catalog)**

### graph.type(type)

**[⇪ Catalog](#catalog)**

* Description: set type: either 'query' or 'shape', and then return a new `graph` object, will decide the query finally goes to `/query/gizmo` or '/shape/gizmo'.

* **type**: either `query` or `shape`.

* Usage examples:

  ```javascript
  // Default to 'query'.
  g.V().All().then((res) => {
    // Your data in JSON.
  }).catch((err) => {
    // Error ...
  });

  // Change to 'shape'
  g.type('shape').V().All().then((res) => {
    // Your data in JSON.
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### graph.Vertex([nodeId])

**[⇪ Catalog](#catalog)**

* Alias: `V`

* Description: starts a query path at the given vertex/vertices, no ids means 'all vertices', return `Query Object`.

* **nodeId**: `optional`, a single string `nodeId` or an array of string `nodeId` represents the starting vertices.

* Usage examples:

  ```javascript
  g.V('</user/shortid/23TplPdS>').All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/23TplPdS>'}]}
  }).catch((err) => {
    // Error ...
  });

  g.type('shape').V(['</user/shortid/23TplPdS>', '</user/shortid/46Juzcyx>']).All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/23TplPdS>'},{id:'</user/shortid/46Juzcyx>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### graph.Morphism()

**[⇪ Catalog](#catalog)**

* Alias: `M`

* Description: creates a morphism `Path` object, unqueryable on it's own, defines one path set in the graph, saving it into a variable then in the future reuse it by using the below `path.Follow(morphism)` and `path.FollowR(morphism)` APIs is the common use case.

* Lets make it more clear by going through the below examples:

  * Lets say we have a query which is getting the `email` and `mobilePhone` of those female followers who follows the one I am following, then we can simply fulfill this requirement by using the below code:

    ```javascript
    g.V('</user/shortid/46Juzcyx>')
      .Out('<follows>').In('<follows>').Has('<gender>', 'F').Out(['<email>', '<mobilePhone>'])
      .All().then((res) => {
        // res will be:
        //   {result:[{id:'_:C'},{id:'xxx.l30@xxx.com'}]}
      }).catch((err) => {
        // Error ...
      });
    ```

  * But because of this query is highly required, we need this piece of query in a lot of other functionalities, we don't want to and shouldn't repeat the code again and again, then we can use `graph.Morphism()` to store this path set into one variable, then at the places we need it, we just reuse it by using the `path.Follow(morphism)` and `path.FollowR(morphism)` APIs, like below:

    ```javascript
    // Then we can reuse this path set later.
    const popularQuery = g.M().Out('<follows>').In('<follows>')
                          .Has('<gender>', 'F').Out(['<email>', '<mobilePhone>']);

    g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).All().then((res) => {
      // res will be exactly same with above query.
      // res will be:
      //   {result:[{id:'_:C'},{id:'xxx.l30@xxx.com'}]}
    }).catch((err) => {
      // Error ...
    });
    ```

**[⇪ Catalog](#catalog)**

### graph.Emit(data)

> Only can be used in the below [gizmoFunc](#gizmo-apis--query-object).

* Description: adds data programmatically to the JSON result list. Can be any JSON type.

* **data**: `required`, a Javascript object that can be serialized to JSON.

**[⇪ Catalog](#catalog)**

## Gizmo APIs → Path Object

> Both `.Morphism()/.M()` and `.Vertex()/.V()` create `Path` object, which provides the following traversal methods.

> Note: that `.Vertex()/.V()` returns a `Query` object, which is a subclass of `Path` object.

> Depends on your `promisify` setting, this lib will provide `bluebird Promise style` or `callback style` API, for both styles, usages examples can be found in the lib [test folder](./test).

**[⇪ Catalog](#catalog)**

### path.Out([predicatePath], [tag])

**[⇪ Catalog](#catalog)**

* Description: `Out()` is the work-a-day way to get between nodes, in the forward direction. Starting with the nodes in path on the subject, follow the quads with predicates defined by **predicatePath** to their objects.

* **predicatePath**: `optional`, one of:

  * null or undefined: All predicates pointing out from this node.
  * a string: The predicate name to follow out from this node.
  * an array of strings: The predicates to follow out from this node.
  * a query path object: The target of which is a set of predicates to follow.

* **tag**: `optional`, one of:

  * null or undefined: No tags.
  * a string: A single tag to add the predicate used to the output set.
  * an array of strings: Multiple tags to use as keys to save the predicate used to the output set.

* Usage examples:

  ```javascript
  g.V('</user/shortid/46Juzcyx>').Out('<follows>', 'predicate').All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/23TplPdS>',predicate:'<follows>'}]}
  }).catch((err) => {
    // Error ...
  });

  // If we pass in one 'predicatePath', but two tags, what the result will look like?
  // As you can see the below result, that indicates: 
  //   every 'predicatePath' just own all the 'tags' you passed in.
  g.V('</user/shortid/46Juzcyx>').Out('<follows>', ['predicate', 'ifExtraTag?']).All().then((res) => {
    // res will be:
    //   {result:[{id:'</user/shortid/23TplPdS>','ifExtraTag?':'<follows>',predicate:'<follows>'}]}
  }).catch((err) => {
    // Error ...
  });

  // Haven't found out one very useful use case here, I mean by passing in a query.
  g.V('</user/shortid/46Juzcyx>').Out(g.V(['<userId>', '<userSetId>']), 'predicate')
    .All().then((res) => {
      // res will be:
      //   {result:[{id:'46Juzcyx',predicate:'<userId>'},{id:'XXX_L14',predicate:'<userSetId>'}]}
    }).catch((err) => {
      // Error ...
    });
  ```

**[⇪ Catalog](#catalog)**

### path.In([predicatePath], [tag])

**[⇪ Catalog](#catalog)**

* Description: same as `Out()`, but in the other direction. Starting with the nodes in path on the object, follow the quads with predicates defined by **predicatePath** to their subjects.

* **predicatePath**: `optional`, one of:

  * null or undefined: All predicates pointing into this node.
  * a string: The predicate name to follow into this node.
  * an array of strings: The predicates to follow into this node.
  * a query path object: The target of which is a set of predicates to follow.

* **tag**: `optional`, one of:

  * null or undefined: No tags.
  * a string: A single tag to add the predicate used to the output set.
  * an array of strings: Multiple tags to use as keys to save the predicate used to the output set.

* Usage examples:

  ```javascript
  g.V('false').In('<isEmailVerified>', 'predicate').All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/46Juzcyx>',predicate:'<isEmailVerified>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Both([predicatePath], [tag])

**[⇪ Catalog](#catalog)**

* Description: same as `In()` and `Out()`, but follow the predicate in either direction(into and out) from the node.

> Note: less efficient, for the moment, as it's implemented with an Or, but useful where necessary.

* **predicatePath**: `optional`, one of:

  * null or undefined: All predicates pointing both into and out from this node.
  * a string: The predicate name to follow both into and out from this node.
  * an array of strings: The predicates to follow both into and out from this node.
  * a query path object: The target of which is a set of predicates to follow.

* **tag**: `optional`, one of:

  * null or undefined: No tags.
  * a string: A single tag to add the predicate used to the output set.
  * an array of strings: Multiple tags to use as keys to save the predicate used to the output set.

* Usage examples:

  ```javascript
  g.V('</user/shortid/BJg4Kj2HOe>').Both('<follows>', 'predicate').All().then((res) => {
    // res will be:
    //   {result:[{id:'</user/shortid/23TplPdS>',predicate:'<follows>'},{id:'</user/shortid/46Juzcyx>',predicate:'<follows>'},{id:'</user/shortid/46Juzcyx>',predicate:'<follows>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Is([node])

**[⇪ Catalog](#catalog)**

* Description: filter all paths to ones, which at this point, are on the given node.

* **node**: `required`, a single string represents a node or an array of string represents a node array.

* Usage examples:

  ```javascript
  g.V('</user/shortid/23TplPdS>').In('<follows>').Is('</user/shortid/46Juzcyx>').All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/46Juzcyx>'}]}
  }).catch((err) => {
    // Error ...
  });

  g.V('</user/shortid/23TplPdS>').In('<follows>')
    .Is(['</user/shortid/46Juzcyx>', '</user/shortid/hwX6aOr7>', '</user/fake/value>'])
    .All().then((res) => {
      // res will be: {result:[{id:'</user/shortid/46Juzcyx>'},{id:'</user/shortid/hwX6aOr7>'}]}
    }).catch((err) => {
      // Error ...
    });
  ```

**[⇪ Catalog](#catalog)**

### path.Has(predicatePath, node)

**[⇪ Catalog](#catalog)**

* Description: 

  * Filter all paths which are, at this point, on the subject for the given predicate and object, but do not follow the path, merely filter the possible paths.
  * Usually useful for starting with all nodes, or limiting to a subset depending on some predicate/value pair.

* **predicatePath**: `required`, a string predicate path.

* **node**: `required`, a string represents a node.

* Usage example:

  ```javascript
  g.V().Has('<gender>', 'F').All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/hwX6aOr7>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.LabelContext([labelPath], [tag])

**[⇪ Catalog](#catalog)**

> Note: the subgraph label of the N-Quads item should always be set to the same value with the the node which it is pointing out from.

* Description: sets (or removes) the subgraph context to consider in the succedent traversals. Affects all `In()`, `Out()`, and `Both()` calls that follow it.

* **labelPath**: `optional`, one of:

  * null or undefined: In future traversals, consider all edges, regardless of subgraph.
  * a string: The name of the subgraph to restrict traversals to.
  * an array of strings: A set of subgraphs to restrict traversals to.
  * a query path object: The target of which is a set of subgraphs.

* **tag**: `optional`, one of:

  * null or undefined: No tags.
  * a string: A single tag to add the last traversed label to the output set.
  * an array of strings: Multiple tags to use as keys to save the label used to the output set.

* Usage examples:

  ```javascript
  g.V('</user/shortid/46Juzcyx>').LabelContext('companyA', 'label').In('<follows>')
    .All().then((res) => {
      // res will be: { result: [ { id: '</user/shortid/23TplPdS>', label: 'companyA' } ] }
    }).catch((err) => {
      // Error ...
    });

  // Then restrict the traversals only to 'companyB' subgraph.
  g.V('</user/shortid/BJg4Kj2HOe>').LabelContext('companyB', 'label').In('<follows>')
    .All().then((res) => {
      // res will be: {result:null}
    }).catch((err) => {
      // Error ...
    });
  ```

**[⇪ Catalog](#catalog)**

### path.Limit(limit)

**[⇪ Catalog](#catalog)**

* Description: limits a number of nodes for current path.

* **limit**: `required`, `Integer`, a number of nodes to limit results to.

* Usage examples:

  ```javascript
  g.V('M').In('<gender>').Limit(1).All().then((res) => {
    // res will be: {result:[{id:'</user/shortid/23TplPdS>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Skip(offset)

**[⇪ Catalog](#catalog)**

* Description: skips a number of nodes for current path.

* **offset**: `required`, `Integer`, a number of nodes to skip.

* Usage examples:

  ```javascript
  // Skip 0.
  g.V().Has('<alpha3CountryCode>', 'SGP').Skip(0).All().then((res) => {
    // res will be:
    //   {result:[{id:'_:A'},{id:'_:B'},{id:'_:C'}]}
  }).catch((err) => {
    // Error ...
  });

  // Skip 2.
  g.V().Has('<alpha3CountryCode>', 'SGP').Skip(2).All().then((res) => {
    // res will be: { result: [ { id: '_:C' } ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.InPredicates()

**[⇪ Catalog](#catalog)**

* Description: get the list of predicates that are pointing into a node.

* Usage examples:

  ```javascript
  g.V('</user/shortid/BJg4Kj2HOe>').InPredicates().All().then((res) => {
    // res will be: { result: [ { id: '<follows>' } ] }
  }).catch((err) => {
    // Error ...
  });

  g.V('true').InPredicates().All().then((res) => {
    // res will be: { result: [ { id: '<isEmailVerified>' }, { id: '<isVerified>' } ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.OutPredicates()

**[⇪ Catalog](#catalog)**

* Description: get the list of predicates that are pointing out from a node.

* Usage example:

  ```javascript
  g.V('</user/shortid/BJg4Kj2HOe>').OutPredicates().All().then((res) => {
    // res will be: {result:[{id:'<userId>'},{id:'<realName>'},{id:'<email>'},{id:'<follows>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Tag([tag])

**[⇪ Catalog](#catalog)**

* Alias: `path.As`

* Description: in order to save your work or learn more about how a path got to the end, you can use this API. The simplest thing to do is to add a tag anywhere you'd like to put each node in the result set.

* **tag**: `required`, a string or an array of strings to act as a result key. The value for tag was the vertex the path was on at the time it reached `Tag([tags])`.

* Usage example:

  ```javascript
  // Find out who is following the peoples I am following.
  g.V('</user/shortid/46Juzcyx>').Tag('step_0').Out('<follows>').Tag('step_1')
    .In('<follows>').Tag('step_2').All((err, res) => {
      // res will be: {result:[{id:'</user/shortid/46Juzcyx>',step_0:'</user/shortid/46Juzcyx>',step_1:'</user/shortid/23TplPdS>',step_2:'</user/shortid/46Juzcyx>'},{id:'</user/shortid/hwX6aOr7>',step_0:'</user/shortid/46Juzcyx>',step_1:'</user/shortid/23TplPdS>',step_2:'</user/shortid/hwX6aOr7>'},{id:'</user/shortid/23TplPdS>',step_0:'</user/shortid/46Juzcyx>',step_1:'</user/shortid/BJg4Kj2HOe>',step_2:'</user/shortid/23TplPdS>'},{id:'</user/shortid/46Juzcyx>',step_0:'</user/shortid/46Juzcyx>',step_1:'</user/shortid/BJg4Kj2HOe>',step_2:'</user/shortid/46Juzcyx>'}]}
    }).catch((err) => {
      // Error ...
    });
  ```

**[⇪ Catalog](#catalog)**

### path.Back(tag)

**[⇪ Catalog](#catalog)**

* Description: ff still valid, a path will now consider their vertex to be the same one as the previously tagged one, with the added constraint that it was valid all the way here, usually will be useful for the below two situations:
  
  * Access intermediate results.
  * Traverse back in queries and take another route for things that have matched so far.

* **tag**: `required`, a previous tag in the query to jump back to.

* Usage example:

  ```javascript
  // Access intermediate results.
  g.V('</user/shortid/46Juzcyx>').Out('<follows>').Tag('myFollowees')
    .In('<follows>').Has('<gender>', 'F')
    .Back('myFollowees').All().then((res) => {
      // res will be:
      //   {result:[{id:'</user/shortid/23TplPdS>',myFollowees:'</user/shortid/23TplPdS>'}]}
      // Try without using 'Back' what will u get.
    }).catch((err) => {
      // Error ...
    });

  // Traverse back in queries and take another route for things that have matched so far.
  g.V('</user/shortid/46Juzcyx>').Out('<follows>').Tag('myFollowees')
    .In('<follows>').Has('<gender>', 'F').Back('myFollowees')
    .Out('<mobilePhone>').Tag('mobilePhone').All().then((res) => {
      // res will be:
      //   {result:[{id:'_:A',mobilePhone:'_:A',myFollowees:'</user/shortid/23TplPdS>'}]}
    }).catch((err) => {
      // Error ...
    });
  ```

**[⇪ Catalog](#catalog)**

### path.Save(predicate, tag)

**[⇪ Catalog](#catalog)**

* Description: from the current node as the subject, save the object of all quads with predicate into tag, without traversal.

* **predicate**: `required`, a string for a predicate node.

* **tag**: a string for a tag key to store the object node.

* Usage example:

  ```javascript
  g.V('</user/shortid/BJg4Kj2HOe>').Save('<follows>', 'target').All().then((res) => {
    // res will be:
    //   {result:[{id:'</user/shortid/BJg4Kj2HOe>',target:'</user/shortid/46Juzcyx>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Intersect(query)

**[⇪ Catalog](#catalog)**

* Alias: `path.And`

* Description: filters all paths by the result of another query path (efficiently computed). This is essentially a join where, at the stage of each path, a node is shared.

* **query**: `required`, another query path, the result sets of which will be intersected.

* Usage example:

  ```javascript
  const queryA = g.V('</user/shortid/46Juzcyx>').Out('<follows>');
  const queryB = g.V('</user/shortid/hwX6aOr7>').Out('<follows>');

  queryA.Intersect(queryB).All().then((res) => {
    // res will be: { result: [ { id: '</user/shortid/23TplPdS>' } ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Union(query)

**[⇪ Catalog](#catalog)**

* Alias: `path.Or`

* Description:

  * Given two queries, returns the combined paths of the two queries.

  * Notice that it's per-path, not per-node.

  * Once again, if multiple paths reach the same destination, they might have different ways of getting there, you can use the `path.Tag([tag])` API to track that.

* **query**: `required`, another query path, the result sets of which will form a union.

* Usage example:

  ```javascript
  // Query the user who has female followers.
  const queryA = g.V().Tag('user').In('<follows>').Has('<gender>', 'F').Back('user');

  // Quer the user whose mobile phone number is verified.
  const queryB = g.V().Tag('user').Out('<mobilePhone>')
                  .Out('<isVerified>').Tag('mobileNoVerified').Is('true');

  // Union above two querys.
  queryA.Union(queryB).All().then((res) => {
    // res will be:
    //   {result:[{id:'</user/shortid/23TplPdS>'},{id:'</user/shortid/46Juzcyx>'},{id:'true',mobileNoVerified:'true',user:'</user/shortid/hwX6aOr7>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Except(query)

**[⇪ Catalog](#catalog)**

* Alias: `path.Difference`

* Description:

  * Removes all paths which match query from path.

  * In a set-theoretic sense, this is (A - B).

  * While `g.V().Except(path)` to achieve `U - B = !B` is supported, it's often very slow.

* **query**: `required`, another query path, the result sets of which will be intersected and negated.

* Usage example:

  ```javascript
  const queryA = g.V('</user/shortid/23TplPdS>').In('<follows>');
  const queryB = g.V('</user/shortid/23TplPdS>').In('<follows>').Has('<gender>', 'M');

  // Query all followers of '</user/shortid/23TplPdS>', except the male one.
  queryA.Except(queryB).All().then((res) => {
    // res will be: { result: [ { id: '</user/shortid/hwX6aOr7>' } ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### path.Follow(morphism)

**[⇪ Catalog](#catalog)**

* Description: 

  * With `graph.Morphism` we can prepare a path for later reuse. `Follow` is the way that's accomplished. Applies the path chain on the morphism object to the current path.

  * Starts as if at the g.M() and follows through the morphism path.

* Usage example: refer to above [graph.Morphism()](#graphmorphism) doc.

**[⇪ Catalog](#catalog)**

### path.FollowR(morphism)

**[⇪ Catalog](#catalog)**

* Description:

  * Same as `Follow` but follows the chain in the reverse direction. Flips "In" and "Out" where appropriate, the net result being a virtual predicate followed in the reverse direction.

  * Starts at the end of the morphism and follows it backwards (with appropriate flipped directions) to the g.M() location.

* Usage example:

  ```javascript
  const willBeReversedQuery = g.M().Out('<follows>');
  
  g.V('</user/shortid/23TplPdS>').FollowR(willBeReversedQuery).All().then((res) => {
    // res will be:
    //   {result:[{id:'</user/shortid/46Juzcyx>'},{id:'</user/shortid/hwX6aOr7>'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

## Gizmo APIs → Query Object

> Subclass of `Path Object`.

> Only `.Vertex()/V()` objects -- that is, queries that have somewhere to start -- can be turned into queries. To actually execute the queries, an output step must be applied.

> Depends on your `promisify` setting, this lib will provide `bluebird Promise style` or `callback style` API, for both styles, usages examples can be found in the lib [test folder](./test).

**[⇪ Catalog](#catalog)**

**Note:**

For the below seven APIs which belong to `Query Object`, the parameter:

  * **callback**: means to provide you a way to receive the error and response when you choose the **callback style APIs**:

    ```javascript
    g.V().All((err, res) => {
      // Deal with the 'err' and 'res' here.
    })
    ```
    , which Cayley/Gizmo doesn't need it. If you set the `promisify` to `true`, then just:

    ```javascript
    g.V().All().then((res) => {
      // Deal with the 'res' here.
    }).catch((err) => {
      // Deal with the 'err' here.
    });
    ```

  * **gizmoFunc**: is a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, try to understand the design here, the **gizmoFunc** should satisfy the following restrictions:
  
    1. No any reference to anything outside of this function, only pure js code.

      * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

      * ES6 'arrow function' and other advanced features cannot be used.
      
    2. Can use any API exposed by this lib belong to 'Path' object.

    3. Can use above [graph.Emit(data)](#graphemitdata) API.

  * Anyway, for each API, you always have lots of wroking well examples can follow, you can find them in this README guidebook or [test folder](./test).

**[⇪ Catalog](#catalog)**

### query.All(callback)

* Description: executes the query and adds the results, with all tags, as a string-to-string (tag to node) map in the output set, one for each path that a traversal could take.

**[⇪ Catalog](#catalog)**

### query.GetLimit(size, callback)

* Description: same as all, but limited to the first `size` unique nodes at the end of the path, and each of their possible traversals.

* **size**, `required`, `integer`, an integer value on the first `size` paths to return.

* Usage example:

  ```javascript
  g.V().GetLimit(1).then((res) => {
    // res will be random.
    // One possible res is: { result: [ { id: '+6586720011' } ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### query.ToArray(gizmoFunc, callback)

**[⇪ Catalog](#catalog)**

* Description: executes a query and returns the results as array at the end of the query path.

* **gizmoFunc**: a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, need to satisfy the below restrictions:

  1. No any reference to anything outside of this function, only pure js code.

    * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

    * ES6 'arrow function' and other advanced features cannot be used.
    
  2. Can use any API exposed by this lib belong to 'Path' object.

  3. Can use above [graph.Emit(data)](#graphemitdata) API.

* Usage example:

  ```javascript
  const popularQuery = g.M().Out('<follows>').In('<follows>')
                        .Has('<gender>', 'F').Out(['<email>', '<mobilePhone>']);

  // Compare with the examples in above 'graph.Morphism()' for better understanding
  // what this API is designed for.
  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).ToArray(function(data) {
    for (var item in data) {
      g.Emit(data[item]);
    }
  }).then((res) => {
    // res will be:
    //   {result:['xxx.l30@xxx.com','_:C']}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### query.TagArray(gizmoFunc, callback)

**[⇪ Catalog](#catalog)**

* Description:

  * Like `.ToArray` above, but instead of a list of top-level strings, returns an Array of tag-to-string dictionaries.

  * Much like `.All` would, except this will be happening inside the Javascript environment.

* **gizmoFunc**: a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, need to satisfy the below restrictions:

  1. No any reference to anything outside of this function, only pure js code.

    * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

    * ES6 'arrow function' and other advanced features cannot be used.
    
  2. Can use any API exposed by this lib belong to 'Path' object.

  3. Can use above [graph.Emit(data)](#graphemitdata) API.

* Usage example:

  ```javascript
  const popularQuery = g.M().Out('<follows>').In('<follows>')
                        .Has('<gender>', 'F').Out('<mobilePhone>').Tag('mobilePhone');
                        
  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).TagArray(function(data) {
    for (var item in data) {
        g.Emit(data[item]);
      }
  }).then((res) => {
    // res will be:
    //   {result:[{id:'_:C',mobilePhone:'_:C'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### query.ToValue(gizmoFunc, callback)

**[⇪ Catalog](#catalog)**

* Description: like `.ToArray` above, but limited to one result node -- a string. Like `.Limit(1)`.

  > Totally cannot get the design point here: like `limit(1)`? cut the results? why?

* **gizmoFunc**: a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, need to satisfy the below restrictions:

  1. No any reference to anything outside of this function, only pure js code.

    * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

    * ES6 'arrow function' and other advanced features cannot be used.
    
  2. Can use any API exposed by this lib belong to 'Path' object.

  3. Can use above [graph.Emit(data)](#graphemitdata) API.

* Usage example:

  ```javascript
  const popularQuery = g.M().Out('<follows>').In('<follows>')
                        .Has('<gender>', 'F').Out(['<email>', '<mobilePhone>']);

  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).ToArray(function(data) {
    g.Emit(data);
  }).then((res) => {
    // res will be:
    //   { result: [ 'xxx.l30@xxx.com' ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### query.TagValue(gizmoFunc, callback)

**[⇪ Catalog](#catalog)**

* Description: like `.TagArray` above, but limited to one result node -- a string. Like `.Limit(1)`. Returns a tag-to-string map.

  > Totally cannot get the design point here: like `limit(1)`? cut the results? why?

* **gizmoFunc**: a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, need to satisfy the below restrictions:

  1. No any reference to anything outside of this function, only pure js code.

    * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

    * ES6 'arrow function' and other advanced features cannot be used.
    
  2. Can use any API exposed by this lib belong to 'Path' object.

  3. Can use above [graph.Emit(data)](#graphemitdata) API.

* Usage example:

  ```javascript
  const popularQuery = g.M().Out('<follows>').In('<follows>')
                        .Has('<gender>', 'F').Out('<mobilePhone>').Tag('mobilePhone');

  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).TagValue(function(data) {
    g.Emit(data);
  }).then((res) => {
    // res will be:
    //   {result:[{id:'_:C',mobilePhone:'_:C'}]}
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

### query.ForEach(limit, gizmoFunc, callback)

**[⇪ Catalog](#catalog)**

* Alias: `query.Map`

* Description: for each tag-to-string result retrieved, as in the `All` case, calls `gizmoFunc(data)` where `data` is the tag-to-string map.

* **limit**: `optional`, `integer`, an integer value on the first limit paths to process.

* **gizmoFunc**: a javascript function which will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine, need to satisfy the below restrictions:

  1. No any reference to anything outside of this function, only pure js code.

    * Coz this function will be serialized and submitted to Cayley/Gizmo server, and then get executed there by the Gizmo JS engine.

    * ES6 'arrow function' and other advanced features cannot be used.
    
  2. Can use any API exposed by this lib belong to 'Path' object.

  3. Can use above [graph.Emit(data)](#graphemitdata) API.

* Usage example:

  ```javascript
  const popularQuery = g.M().Out('<follows>').In('<follows>')
                        .Has('<gender>', 'F').Out(['<email>', '<mobilePhone>']);

  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).ForEach(function(data) {
    for (var item in data) {
      g.Emit(data[item]);
    }
  }).then((res) => {
    // res will be:
    //   {result:['xxx.l30@xxx.com','_:C']}
  }).catch((err) => {
    // Error ...
  });

  g.V('</user/shortid/46Juzcyx>').Follow(popularQuery).ForEach(1, function(data) {
    for (var item in data) {
      g.Emit(data[item]);
    }
  }).then((res) => {
    // res will be: { result: [ 'xxx.l30@xxx.com' ] }
  }).catch((err) => {
    // Error ...
  });
  ```

**[⇪ Catalog](#catalog)**

## Additional resources

<a href="https://github.com/lnshi/node-cayley/blob/master/ADDITIONAL_RESOURCES.md" target="_blank">See here.</a>


