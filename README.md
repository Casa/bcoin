# Bcoin

[![Build Status][circleci-status-img]][circleci-status-url]
[![Coverage Status][coverage-status-img]][coverage-status-url]

**Bcoin** is an alternative implementation of the Bitcoin protocol, written in
JavaScript and C/C++ for Node.js.

Bcoin is well tested and aware of all known consensus rules. It is currently
used in production as the consensus backend and wallet system for
[purse.io][purse].


## Uses

- Full Node
- SPV Node
- Wallet Backend
- Mining Backend (getblocktemplate support)
- Layer 2 Backend (lightning)
- General Purpose Bitcoin Library

Try it in the browser: [https://bcoin.io/browser/](https://bcoin.io/browser/)

## Install

```
$ git clone git://github.com/bcoin-org/bcoin.git
$ cd bcoin
$ npm rebuild
$ ./bin/bcoin
```

See the [Getting started][guide] guide for more in-depth installation
instructions, including verifying releases. If you're upgrading, see the
latest changes via the [Changelog][changelog].

## Build for iOS

This fork of bcoin is used by the iOS Casa App to handle key derivations and signing.

Note! Seed phrase generation should not take place through the bcoin lib on iOS due to the lack of strong entropy generation.

### Compiling

```
npm install
npm install bpkg
bpkg --browser --name bcoin --umd --output browser-new/bcoin-2.js ../bcoin-browser.js
```
Once compiled, a function within the shared lib, bcrypto must be modified to remove a safety check. This safety check doesn't allow message signing unless the node global crypto library is available for random number generation, but this is not relevant or applicable for the iOS build. 

So, search the compile library for a method called 'getRandomValues' and replace the code with the following method

```
/// OLD METHOD
function getRandomValues(array) {
  if (!HAS_CRYPTO)
    throw new Error('Entropy source not available.');

  return randomValues(array);
}

/// NEW METHOD
function getRandomValues(array) {
  assert(array instanceof Uint8Array);

  if (array.length > (2 ** 31 - 1))
    throw new RangeError('The value "size" is out of range.');

  const crypto = global.crypto || global.msCrypto;

  // Native WebCrypto support.
  // https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues
  if (crypto && typeof crypto.getRandomValues === 'function') {
    const max = 65536;

    if (array.length > max) {
      for (let i = 0; i < array.length; i += max) {
        let j = i + max;

        if (j > array.length)
          j = array.length;

        crypto.getRandomValues(array.subarray(i, j));
      }
    } else {
      if (array.length > 0)
        crypto.getRandomValues(array);
    }

    return;
  }

  // Fallback to Math.random 
  for (let i = 0; i < array.length; i++)
    array[i] = Math.floor(Math.random() * 256);
  return;
}
```

```
npm install uglify-js
uglifyjs ../bcoin.js -o ../bcoin-min.js
```

## Documentation

- General docs: [docs/](docs/README.md)
- Wallet and node API docs: https://bcoin.io/api-docs/
- Library API docs: https://bcoin.io/docs/

## Support

Join us on [freenode][freenode] in the [#bcoin][irc] channel.

## Disclaimer

Bcoin does not guarantee you against theft or lost funds due to bugs, mishaps,
or your own incompetence. You and you alone are responsible for securing your
money.

## Contribution and License Agreement

If you contribute code to this project, you are implicitly allowing your code
to be distributed under the MIT license. You are also implicitly verifying that
all code is your original work. `</legalese>`

## License

- Copyright (c) 2014-2015, Fedor Indutny (MIT License).
- Copyright (c) 2014-2017, Christopher Jeffrey (MIT License).

See LICENSE for more info.

[purse]: https://purse.io
[guide]: docs/getting-started.md
[freenode]: https://freenode.net/
[irc]: irc://irc.freenode.net/bcoin
[changelog]: CHANGELOG.md

[coverage-status-img]: https://codecov.io/gh/bcoin-org/bcoin/badge.svg?branch=master
[coverage-status-url]: https://codecov.io/gh/bcoin-org/bcoin?branch=master
[circleci-status-img]: https://circleci.com/gh/bcoin-org/bcoin/tree/master.svg?style=shield
[circleci-status-url]: https://circleci.com/gh/bcoin-org/bcoin/tree/master
