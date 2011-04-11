A "json" command for massaging JSON on your Unix command line. This
is a single-file node.js script.


# Installation

1. Get [node](http://nodejs.org).

2. Get the 'json' script and put it on your PATH somewhere. For example:

        cd ~/bin
        curl https://github.com/trentm/json/raw/master/json > json
        chmod 755 json


# Usage

`json -h`:

    Usage: <something generating JSON on stdout> | json [options] [lookup]

    Pipe in your JSON for nicer output. Or supply a `lookup` to extract
    a subset of the JSON. HTTP header blocks are skipped by default
    (as from `curl -i`) by default.

    By default, the output is JSON-y: JSON except for a simple string return
    value, which is printed without quotes. Use '-j' or '-i' to override.

    Options:
      -h, --help  print this help info and exit
      --version   print version of this command and exit
      -H          drop any HTTP header block
      -i          output using node's `sys.inspect`
      -j          output using `JSON.stringfy`, i.e. strict JSON

    Examples:
      curl -s http://search.twitter.com/search.json?q=node.js | json
      curl -s http://search.twitter.com/search.json?q=node.js | json results

    See <https://github.com/trentm/json> for more.


# Examples

Using the Github API to look at [node](https://github/joyent/node):

    $ curl -s http://github.com/api/v2/json/repos/show/joyent/node
    {"repository":{"has_wiki":true,"forks":597,"language":"C++","pushed_at":"2011/03/18 15:19:24 -0700","homepage":"http://nodejs.org/","open_issues":271,"fork":false,"has_issues":true,"url":"https://github.com/joyent/node","created_at":"2009/05/27 09:29:46 -0700","size":20984,"private":false,"has_downloads":false,"name":"node","owner":"joyent","organization":"joyent","watchers":5584,"description":"evented I/O for v8 javascript"}}

Nice output by default:

    $ curl -s http://github.com/api/v2/json/repos/show/joyent/node | json
    {
      "repository": {
        "has_wiki": true,
        "language": "C++",
        "homepage": "http://nodejs.org/",
        "open_issues": 271,
        "fork": false,
        "has_issues": true,
        "url": "https://github.com/joyent/node",
        "forks": 597,
        "created_at": "2009/05/27 09:29:46 -0700",
        "pushed_at": "2011/03/18 15:19:24 -0700",
        "size": 20984,
        "private": false,
        "has_downloads": false,
        "name": "node",
        "owner": "joyent",
        "organization": "joyent",
        "watchers": 5584,
        "description": "evented I/O for v8 javascript"
      }
    }

Say you just want to extract one value:

    $ curl -s https://github.com/api/v2/json/repos/show/joyent/node | json repository.open_issues
    271

If you use `curl -i` to get HTTP headers (because perhaps they contain relevant information), `json` will skip those automatically:

    $ curl -is https://github.com/api/v2/json/repos/show/joyent/node | json repository
    HTTP/1.1 200 OK
    Server: nginx/0.7.67
    Date: Fri, 11 Feb 2011 06:45:02 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Status: 200 OK
    X-RateLimit-Limit: 60
    ETag: "2fcb49428e91d0823acf453b15ccc0b5"
    X-RateLimit-Remaining: 59
    X-Runtime: 13ms
    Content-Length: 398
    Cache-Control: private, max-age=0, must-revalidate

    {
      "forks": 514,
      "has_issues": true,
      "language": "C++",
      "url": "https://github.com/joyent/node",
      "homepage": "http://nodejs.org/",
      "has_downloads": false,
      "watchers": 4753,
      "fork": false,
      "created_at": "2009/05/27 09:29:46 -0700",
      "size": 15316,
      "private": false,
      "has_wiki": true,
      "name": "node",
      "owner": "joyent",
      "pushed_at": "2011/02/10 02:48:07 -0800",
      "description": "evented I/O for v8 javascript",
      "open_issues": 229
    }

Or, say you are stuck with the headers in your pipeline, `json -H` will drop them:

    $ curl -is https://github.com/api/v2/json/repos/show/joyent/node | json -H repository.watchers
    4753

Here is an example that shows indexing a list. (The given "lookup" argument is basically
JavaScript code appended, with '.' if necessary, to the JSON data and eval'd.)

    $ curl -s http://github.com/api/v2/json/repos/search/nodejs | json 'repositories[2].description'
    Connect is a middleware layer for Node.js


# Test suite

    make test

Note that the 'utf8' test fails with a V8 build before  [revision
5399](http://code.google.com/p/v8/source/detail?r=5399), see
<http://code.google.com/p/v8/issues/detail?id=855>. I'm not sure exactly what
node versions this corresponds to, but at least: broken in node 0.2.5 and
working in node 0.4.0.

If you are stuck with an older node and don't want the utf8 failure in your face:

    TEST_ONLY=-utf8 make test



# TODO

- consider a '*' syntax for running the subsequent lookup on all items in a list


# Alternatives you might prefer

- jsonpipe: <https://github.com/dvxhouse/jsonpipe>
- json-command: <https://github.com/zpoley/json-command>
- JSONPath: <http://goessner.net/articles/JsonPath/>, <http://code.google.com/p/jsonpath/wiki/Javascript>
- jsawk: <https://github.com/micha/jsawk>
- jshon: <http://kmkeen.com/jshon/>
