## Simplistic CouchDB client with a minimal level of abstraction

*Explanation by example:*

Simply create a new instance, passing database name (or complex options):

    var db = new exports.Db('blog-posts');
    var slaveDb = new exports.Db({db:'blog-posts', host:'dbslave.local'});

A db object is merely a gateway for dispatching requests to a particular server. The following GETs a document with the key "foo" in the database "blog-posts":

    db.get('foo', function(err, result) {
      if (err) sys.error(err.stack);
      sys.log('document -> '+sys.inspect(result));
    });

The above call is equivalent to this request:

    GET /blog-posts/foo HTTP/1.1
    Host: localhost
    Connection: Keep-Alive

There are very few helper methods, simply because db.get("/action") is
verbose enough and close to the actual CouchDB API. however, some common
functionality is available at a higher level, for instance "_all_docs":

    db.allDocs(["foo", "bar"], function(err, docs) {
      if (err) sys.error(err.stack);
      sys.log('documents -> '+sys.inspect(docs));
    });

The above is roughly equivalent to this lower level call:

    db.post('_all_docs?include_docs=true', {keys:["foo", "bar"]},
      function(err, result)
    {
      if (err) return sys.error(err.stack);
      var docs = {};
      result.rows.forEach(function(row){ docs[row.id] = row.doc||row.value; });
      sys.log('documents -> '+sys.inspect(docs));
    });

Requesting stuff outside of a "database" is possible in multiple ways. a)
By creating an instance without database/namespace. The resulting handle have
no prefix and thus all requests roots themselves at "/". However, as a
convenience mechanism, you can prefix paths with a "/" character to indicate
the path is absolute:

    db.get('/_all_dbs', function(err, result) {
      if (err) sys.error(err.stack);
      sys.log('databases -> '+sys.inspect(result));
    });


All calls are asynchronous. There's a connection pool used in the background,
mainly taking case of two things:

1. keeping ~1 alive connection for low latency in low traffic environments.

2. limiting the number of concurrent requests -- when the limit is hit,
   requests will be held in a queue until a connection becomes available.

If you require monotonicity -- things happening in a sequential order -- just do like you do with all other async calls:

    db.get('/_uuids?count=1', function(err, result){
      if (err) return sys.error(err.stack);
      var _id = result.uuids[0];
      db.put(_id, {"hello": "world"}, function(err, result) {
        if (err) return sys.error(err.stack);
        db.get(_id, function(err, doc) {
          if (err) return sys.error(err.stack);
          sys.log('Created and retrieved '+_id+' -> '+sys.inspect(doc));
        });
      });
    });

> **Looking for something high-level?** -- Check out [node-couchdb](http://github.com/felixge/node-couchdb) which provides a javascript API layer on top of the CouchDB API.

More detailed documentation is available in-line the `couchdb.js` file.

## Requirements

- [node](http://nodejs.org/) `>=0.1.32`

## MIT license

Copyright (c) 2010 Rasmus Andersson <http://hunch.se/>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
