Bch Wallet Client
======

[![NPM Package](https://img.shields.io/npm/v/bch-wallet-client.svg?style=flat-square)](https://www.npmjs.org/package/bch-wallet-client)
[![Build Status](https://img.shields.io/travis/owstack/bch-wallet-client.svg?branch=master&style=flat-square)](https://travis-ci.org/owstack/bch-wallet-client) 
[![Coverage Status](https://coveralls.io/repos/owstack/bch-wallet-client/badge.svg)](https://coveralls.io/r/owstack/bch-wallet-client)

The *official* client library for [bch-wallet-service](https://github.com/owstack/bch-wallet-service).

## Attribution

This repository was created by copy forking [bitcore-wallet-client commit d986bbb](https://github.com/bitpay/bitcore-wallet-client/commit/d986bbb69d01be56f1cfd09c89625f587de2bc02).

## Description

This package communicates with BCHWS [Bch wallet service](https://github.com/owstack/bch-wallet-service) using the REST API. All REST endpoints are wrapped as simple async methods. All relevant responses from BCHWS are checked independently by the peers, thus the importance of using this library when talking to a third party BCHWS instance.

See [Bch-wallet](https://github.com/owstack/bch-wallet) for a simple CLI wallet implementation that relays on BCHWS and uses bch-wallet-client.

## Get Started

You can start using bch-wallet-client in any of these two ways:

* via [Bower](http://bower.io/): by running `bower install bch-wallet-client` from your console
* or via [NPM](https://www.npmjs.com/package/bch-wallet-client): by running `npm install bch-wallet-client` from your console.

## Example

Start your own local [Bch wallet service](https://github.com/owstack/bch-wallet-service) instance. In this example we assume you have `bch-wallet-service` running on your `localhost:3232`.

Then create two files `irene.js` and `tomas.js` with the content below:

**irene.js**

``` javascript
var Client = require('@owstack/bch-wallet-client');


var fs = require('fs');
var BCHWS_INSTANCE_URL = 'https://bchws.openwalletstack.com/bchws/api'

var client = new Client({
  baseUrl: BCHWS_INSTANCE_URL,
  verbose: false,
});

client.createWallet("My Wallet", "Irene", 2, 2, {network: 'testnet'}, function(err, secret) {
  if (err) {
    console.log('error: ',err); 
    return
  };
  // Handle err
  console.log('Wallet Created. Share this secret with your copayers: ' + secret);
  fs.writeFileSync('irene.dat', client.export());
});
```

**tomas.js**

``` javascript

var Client = require('@owstack/bch-wallet-client');


var fs = require('fs');
var BCHWS_INSTANCE_URL = 'https://bchws.openwalletstack.com/bchws/api'

var secret = process.argv[2];
if (!secret) {
  console.log('./tomas.js <Secret>')

  process.exit(0);
}

var client = new Client({
  baseUrl: BCHWS_INSTANCE_URL,
  verbose: false,
});

client.joinWallet(secret, "Tomas", {}, function(err, wallet) {
  if (err) {
    console.log('error: ', err);
    return
  };

  console.log('Joined ' + wallet.name + '!');
  fs.writeFileSync('tomas.dat', client.export());


  client.openWallet(function(err, ret) {
    if (err) {
      console.log('error: ', err);
      return
    };
    console.log('\n\n** Wallet Info', ret); //TODO

    console.log('\n\nCreating first address:', ret); //TODO
    if (ret.wallet.status == 'complete') {
      client.createAddress({}, function(err,addr){
        if (err) {
          console.log('error: ', err);
          return;
        };

        console.log('\nReturn:', addr)
      });
    }
  });
});
```

Install `bch-wallet-client` before start:

```
npm i bch-wallet-client
```

Create a new wallet with the first script:

```
$ node irene.js
info Generating new keys 
 Wallet Created. Share this secret with your copayers: JbTDjtUkvWS4c3mgAtJf4zKyRGzdQzZacfx2S7gRqPLcbeAWaSDEnazFJF6mKbzBvY1ZRwZCbvT
```

Join to this wallet with generated secret:

```
$ node tomas.js JbTDjtUkvWS4c3mgAtJf4zKyRGzdQzZacfx2S7gRqPLcbeAWaSDEnazFJF6mKbzBvY1ZRwZCbvT
Joined My Wallet!

Wallet Info: [...]

Creating first address:

Return: [...]

```

Note that the scripts created two files named `irene.dat` and `tomas.dat`. With these files you can get status, generate addresses, create proposals, sign transactions, etc.


