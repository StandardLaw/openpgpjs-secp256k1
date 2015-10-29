### Description 

This work is to provide an implementation of [RFC 6637](http://www.ietf.org/rfc/rfc6637.txt) to fix [Issue #26: Integrate fast ECC support](https://github.com/openpgpjs/openpgpjs/issues/26) into OpenPGP.js.


### Compatibility

In order to assure compatibility of the provided implementation with RFC 6637, the keys, messages were tested with a latest release version of GnuPG v2.1.8, compiled with a beta version of libgcrypt v1.7.0-beta262.
It was tested that keys, messages, and signatures generated by GnuPG were imported correctly, and also keys, messages and signatures generated by this implementation are correctly managed by GnuPG.

```
> gpg2 --homedir ..\home --version
gpg (GnuPG) 2.1.8
libgcrypt 1.7.0-beta262
Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: ../home
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB
```

### New Dependencies

There are two new dependencies:
* [Elliptic](https://github.com/indutny/elliptic/) for the elliptic curve cryptography. MIT license.
* [Aes](https://github.com/cryptocoinjs/aes) required to implement RFC 3394 Key wrap and Key Unwrap functions. BSD License.


### Examples

#### Generate new keypair
```js
var openpgp = require('openpgp');

var options = {
    curve: 'secp256k1',
    userId: 'Hamlet <hamlet@example.net>',
    passphrase: 'To be, or not to be: that is the question'
};

openpgp.generateKeyPair(options).then(function(keypair) {
    // success
    var privkey = keypair.privateKeyArmored;
    var pubkey = keypair.publicKeyArmored;
}).catch(function(error) {
    // failure
});
```

#### Generate keypair from bitcoin key
```js
var openpgp = require('openpgp');
var bs58check = require('bs58check');

var wif = 'KyiAchQgMKuXQu89j6k6UVZQj7brK6cM79JfmDvkNXPVW24L1thi';
var buff = bs58check.decode(wif);
var privateKey = buff.slice(1, -1);
privateKey = openpgp.util.bin2str(privateKey);

var options = {
    curve: 'secp256k1',
    userId: 'Hamlet <hamlet@example.net>',
    passphrase: 'To be, or not to be: that is the question',
    material {
      key: privateKey,
      subkey: privateKey
    }
};

openpgp.generateKeyPair(options).then(function(keypair) {
    // success
    var privkey = keypair.privateKeyArmored;
    var pubkey = keypair.publicKeyArmored;
}).catch(function(error) {
    // failure
});
```

#### Signature, encryption and decryption
The normal operations: signature, encryption and decryption require no modifications.
```js
var openpgp = require('openpgp');

var key = '-----BEGIN PGP PUBLIC KEY BLOCK ... END PGP PUBLIC KEY BLOCK-----';
var publicKey = openpgp.key.readArmored(key);

openpgp.encryptMessage(publicKey.keys, 'Hello, World!').then(function(pgpMessage) {
    // success
}).catch(function(error) {
    // failure
});
```

### Improvements

* The dependency with AES library can be eliminated, if the OpenPGP.js crypto/cipher/aes.js is modified to provide a decrypt function. It is only used by the wrap and unwrap functions in the crypto/rfc3394.js file.


### Note

Althought the example uses the same value to generate the main key, and the subkey, it is a recommended practice to use different keys. The main key is used for signature and the subkeys are used for encryption.


### Resources

* Elliptic Curve Cryptography (ECC) in OpenPGP [RFC 6637](http://www.ietf.org/rfc/rfc6637.txt)
* OpenPGP Message Format [RFC 4880](http://www.ietf.org/rfc/rfc4880.txt)
* Advanced Encryption Standard (AES) Key Wrap Algorithm [RFC 3394](http://www.ietf.org/rfc/rfc3394.txt)
* A JavaScript component for the Advanced Encryption Standard (AES) [AES](https://github.com/cryptocoinjs/aes)
* Fast elliptic-curve cryptography in a plain javascript implementation [Elliptic](https://github.com/indutny/elliptic/)
