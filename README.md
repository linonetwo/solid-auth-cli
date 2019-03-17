# solid-auth-cli
**a node/command-line Solid client with persistent login**
<br><a href="http://badge.fury.io/js/solid-auth-cli">![npm](https://badge.fury.io/js/solid-auth-cli.svg)</a>

This library supports login and persistent connection to Solid from command-line and node apps. It (and apps using it) can make authorized fetches either directly using the fetch and REST APIs, or using rdflib's fetcher.

### Use with rdflib.js

You must be using Version 0.19.1 or later of rdflib.js to use it in conjunction with solid-auth-cli.  And you must initialize your fetcher as shown below.

  ```javascript
  const $rdf = require('rdflib');
  const auth = require('solid-auth-cli');
  const store = $rdf.graph();
  const fetcher = $rdf.fetcher( store, {fetch:auth.fetch} );
  auth.login().then( session => {
      fetcher.load( ... ).then( response => {
          // ...
      }, console.log() )
  }, console.log() )
  ```  


### Wrting scripts that work in both the browser and node

Solid-auth-cli is built on top of solid-cli, providing persistance and the same API as solid-auth-client. Since it mimics all of solid-auth-client's methods, scripts can work in both browserless and browser contexts by switching which library is called like this:

  if(typeof window === "undefined"){
      solid = { auth:require('solid-auth-cli') }
  }
  solid.auth.login()... // this will now use solid-auth-client in the browser and solid-auth-cli in node



### login()
### login( path-to-credentials-json-file )
### login({ idp:"https://idp.example.com", username:"you", password:"hmm" })

The login method needs an Identity Provider, username, and password.  Those may be passed in as an object or read from a specified JSON file.  If called with no arguments, login() will look for a configuration file in ~/.solid-auth-cli-config.json. If there is no configuration file or it is missing an IDP, username, or password, you will be prompted to enter them when you login.


### An example
```javascript
const solid = { auth:require('solid-auth-cli') }
const resource = "https://me.example.com/private/hidden.ttl"
const expected = "only the owner"                                 
const idp = "https://example.com"

console.log("logging in ...")
login().then( session => {
    console.log(`logged in as <${session.webId}>`)
    solid.auth.fetch(resource).then( response => {
        if (!response.ok) console.log(response.status+" "+response.statusText)
        response.text().then( content => {
            if( content.match(new RegExp(expected)) ) console.log("ok")
            else console.log("Got something , but not the right thing.")
        },e => console.log("Error parsing : "+e))
    },e => console.log("Error fetching : "+e))
},e => console.log("Error logging in : "+e))

async function login() {
    var session = await solid.auth.currentSession()
    if (!session) session = await solid.auth.login()
    return session;
}
```

&copy; Jeff Zucker, 2019, may be freely distributed using an MIT license
