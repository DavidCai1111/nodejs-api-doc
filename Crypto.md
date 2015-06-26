# Crypto#

### 稳定度: 2 - 稳定

使用`require('crypto')`来获取这个模块。

`crypto`模块提供了一种封装安全证书的方法，用来作为安全HTTPS网络和HTTP链接的一部分。

它同样也通过了一个OpenSSL hash，hamc，cipher，decipher，sign和vierify方法包装的集合。

#### crypto.setEngine(engine[, flags])#

加载和设置 一些/所有 OpenSSL功能引擎（由标记选择）。

引擎可以通过id或 引擎共享库的路径 来选择。

`flags`是可选的，并且有一个`ENGINE_METHOD_ALL`默认值。可以选一个或多个以下的标记（在常量模块中定义）。

 - ENGINE_METHOD_RSA
 - ENGINE_METHOD_DSA
 - ENGINE_METHOD_DH
 - ENGINE_METHOD_RAND
 - ENGINE_METHOD_ECDH
 - ENGINE_METHOD_ECDSA
 - ENGINE_METHOD_CIPHERS
 - ENGINE_METHOD_DIGESTS
 - ENGINE_METHOD_STORE
 - ENGINE_METHOD_PKEY_METH
 - ENGINE_METHOD_PKEY_ASN1_METH
 - ENGINE_METHOD_ALL
 - ENGINE_METHOD_NONE

#### crypto.getCiphers()#

返回一个支持的加密算法的名字数组。

例子：

```js
var ciphers = crypto.getCiphers();
console.log(ciphers); // ['aes-128-cbc', 'aes-128-ccm', ...]
```

#### crypto.getHashes()#

返回一个支持的哈希算法的名字数组。

例子：

```js
var hashes = crypto.getHashes();
console.log(hashes); // ['sha', 'sha1', 'sha1WithRSAEncryption', ...]
```

#### crypto.getCurves()#

返回一个支持的椭圆加密算法的名字数组。

例子：

```js
var curves = crypto.getCurves();
console.log(curves); // ['secp256k1', 'secp384r1', ...]
```

#### crypto.createCredentials(details)#

 >稳定度: 0 - 弃用。使用`tls.createSecureContext`代替。
 
创建一个加密凭证对象，接受一个可选的带键字典`details`：

 - pfx : 一个带着`PFX`或`PKCS12`加密的私钥，加密凭证和CA证书的字符串或`buffer`。
 - key : 一个带着`PEM`加密私钥的字符串。
 - passphrase : 一个私钥或`pfx`密码字符串。
 - cert : 一个带着`PEM`加密凭证的字符串。
 - ca : 一个用来信任的`PEM`加密CA证书的字符串或字符串列表。
 - crl : 一个`PEM`加密`CRL`的字符串或字符串列表。
 - ciphers: 一个描述需要使用或排除的加密算法的字符串。更多加密算法的格式细节参阅`http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT`

如果没有指定`ca`，那么`io.js`将会使用`http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt.`提供的默认公共可信任`CA`列表。

#### crypto.createHash(algorithm)#

创建并返回一个哈希对象，一个指定算法的加密哈希用来生成哈希摘要。

`algorithm `依赖于平台上的OpenSSL版本所支持的算法。例如`'sha1'`，`'md5'`，`'sha256'`，`'sha512'`等等。`openssl list-message-digest-algorithms`命令会展示可用的摘要算法。

例子：这个程序计算出一个文件的sha1摘要：

```js
var filename = process.argv[2];
var crypto = require('crypto');
var fs = require('fs');

var shasum = crypto.createHash('sha1');

var s = fs.ReadStream(filename);
s.on('data', function(d) {
  shasum.update(d);
});

s.on('end', function() {
  var d = shasum.digest('hex');
  console.log(d + '  ' + filename);
});
```

#### Class: Hash#

这个类用来创建数据哈希摘要。

这是一个同时可读与可写的流。写入的数据用来计算哈希。一旦当流的可写端终止，使用`read()`来获取计算所得哈希摘要。遗留的`update`和`digest`方法同样被支持。

通过`crypto.createHash`返回。

#### hash.update(data[, input_encoding])#

使用给定的`data`更新哈希内容，通过`input_encoding`指定的编码可以是`'utf8'`，`'ascii'`或`'binary'`。如果没有提供编码，并且输入是一个字符串，那么将会指定编码为`'binary'`。如果`data`是一个`Buffer`那么`input_encoding`会被忽略。

它是流式数据，所以这个方法可以被调用多次。

#### hash.digest([encoding])#

计算所有的被传递的数据的摘要。`encoding `可以是`'binary'`，`'hex'`或`'base64'`。如果没有指定编码，那么一个`buffer`被返回。

注意：当调用了`digest()`方法之后，哈希对象不能再被使用了。

#### crypto.createHmac(algorithm, key)#

创建并返回一个hmac对象，即通过给定的算法和密钥生成的加密图谱（cryptographic）。

这是一个既可读又可写的流。写入的数据被用来计算hamc。一旦当流的可写端终止，使用`read()`方法来获取计算所得摘要值。遗留的`update`和`digest`方法同样被支持。

`algorithm `依赖于平台上的OpenSSL版本所支持的算法。参阅上文`createHash`。`key`是要使用的hmac密钥。

#### Class: Hmac#

用于创建hmac加密图谱（cryptographic）的类。

通过`crypto.createHmac`返回。

#### hmac.update(data)#

只用指定的`data`更新hmac内容。因为它是流式数据，所以这个方法可以被调用多次。

#### hmac.digest([encoding])#

计算所有的被传递的数据的hmac摘要。`encoding `可以是`'binary'`，`'hex'`或`'base64'`。如果没有指定编码，那么一个`buffer`被返回。

注意：当调用了`digest()`方法之后，hmac对象不能再被使用了。

#### crypto.createCipher(algorithm, password)#

创建和返回一个`cipher`对象，指定指定的算法和密码。

算法依赖于OpenSSL，如果`'aes192'`，等等。在最近的发行版中，`openssl list-cipher-algorithms`命令会展示可用的`cipher`算法。密码被用来获取密钥和IV，必须是一个`'binary'`编码的字符串或`buffer`。

这是一个既可读又可写的流。写入的数据被用来计算哈希。一旦当流的可写端终止，使用`read()`方法来获取通过`cipher`计算所得的内容。遗留的`update`和`digest`方法同样被支持。

注意：`createCipher`通过 无盐`MD5`一次迭代所得的摘要 来调用 `OpenSSL函数`EVP_BytesToKey` 来派生密钥。无盐意味允许字典攻击，即同样的密码经常可以用来创建同样的密钥。一次迭代并且无加密图谱安全（non-cryptographically secure）以为着允许密码被快速测试。

OpenSSL建议使用`pbkdf2`替代`EVP_BytesToKey`，推荐你通过`crypto.pbkdf2`然后调用`createCipheriv()`创建一个`cipher`流来派生一个密钥和iv。

#### crypto.createCipheriv(algorithm, key, iv)#

创建和返回一个`cipher`对象，指定指定的算法，密钥和iv。

`algorithm`参数与`createCipher()`相同。`key`是被算法使用的源密钥（raw key）。iv是初始化矢量（initialization vector）。

`key`和`iv`必须是`'binary'`编码的字符串或`buffer`。

#### Class: Cipher#

创建一个加密数据。

由`crypto.createCipher`和`crypto.createCipheriv`返回。

这是一个既可读又可写的流。写入的文本数据被用来在可读端生产被加密的数据。遗留的`update`和`final`方法同样被支持。

#### cipher.update(data[, input_encoding][, output_encoding])#

通过`data`更新`cipher`，`input_encoding`中指定的编码可以是`'utf8'`，`'ascii'`或`'binary'`。如果没有提供编码，那么希望接受到一个`buffer`。如果数据是一个`Buffer`，那么`input_encoding`将被忽略。

`output_encoding`指定了加密数据的输出格式，可以是`'binary'`，`'base64'`或`'hex'`。如果没有指定编码，那么一个`buffer`会被返回。

返回一个加密内容，并且因为它是流式数据，所以可以被调用多次。

#### cipher.final([output_encoding])#

返回所有的剩余的加密内容，`output_encoding`可以是`'binary'`，`'base64'`或`'hex'`。如果没有指定编码，那么一个`buffer`会被返回。

注意：当调用了`final()`方法之后，cipher对象不能再被使用了。

#### cipher.setAutoPadding(auto_padding=true)#

你可以禁用自动填充输入数据至块大小。如果`auto_padding`为`false`，那么整个输入数据的长度必须`cipher`的块大小的整数倍，否则会失败。这对非标准填充非常有用，如使用0x0替代PKCS填充。你必须在`cipher.final`之前调用它。

#### cipher.getAuthTag()#

对于已认证加密模式（当前支持：GCM），这个方法返回一个从给定数据计算所得的代表了认证标签的`Buffer`。必须在`final`方法被调用后调用。

#### cipher.setAAD(buffer)#

对于已认证加密模式（当前支持：GCM），这个方法设置被用于额外已认证数据（AAD）输入参数的值。

#### crypto.createDecipher(algorithm, password)#

使用给定算法和密钥，创建并返回一个解密器对象。这是上文`createCipher()`的一个镜像。

#### crypto.createDecipheriv(algorithm, key, iv)#

使用给定算法，密钥和iv，创建并返回一个解密器对象。这是上文`createCipheriv()`的一个镜像。

#### Class: Decipher#

解密数据类。

通过`crypto.createDecipher`和`crypto.createDecipheriv`返回。

这是一个既可读又可写的流。写入的被加密的数据被用来在可读端生产文本数据。遗留的`update`和`final`方法同样被支持。

#### decipher.update(data[, input_encoding][, output_encoding])#

通过`data`更新`decipher`，编码可以是`'binary'`，`'base64'`或`'hex'`。如果没有提供编码，那么希望接受到一个`buffer`。如果数据是一个`Buffer`，那么`input_encoding`将被忽略。

`output_encoding`指定了解密数据的输出格式，可以是`'binary'`，`'ascii'`或`'utf8'`。如果没有指定编码，那么一个`buffer`会被返回。

#### decipher.final([output_encoding])#

返回所有的剩余的文本数据，`output_encoding`可以是`'binary'`，`'ascii'`或`'utf8'`。如果没有指定编码，那么一个`buffer`会被返回。

注意：当调用了`final()`方法之后，decipher对象不能再被使用了。

#### decipher.setAutoPadding(auto_padding=true)#

如数据没有使用标准块填充阻止`decipher.final`检查和删除它来加密，你可以禁用自动填充。那么整个输入数据的长度必须`cipher`的块大小的整数倍，否则会失败。你必须在将数据导流至`decipher.update`前调用它。

#### decipher.setAuthTag(buffer)#

对于已认证加密模式（当前支持：GCM），这个方法必须被传递，用来接受认证标签。如果没有提供标签或密文被干扰，最终会抛出一个错误。

#### decipher.setAAD(buffer)#

对于已认证加密模式（当前支持：GCM），这个方法设置被用于额外已认证数据（AAD）输入参数的值。

#### crypto.createSign(algorithm)#

使用指定的算法，创建并返回一个数字签名类。在最近的OpenSSL发行版中，`openssl list-public-key-algorithms`会列出所有支持的数字签名算法。例如`'RSA-SHA256'`。

#### Class: Sign#

用于生成数字签名的类。

通过`crypto.createSign`返回。

`Sign`对象是一个可写流。写入的数据用来生成数字签名。一旦所有的数据被写入，`sign`方法会返回一个数字签名。遗留的`update`方法也支持。

#### sign.update(data)#

使用`data`更新`sign`对象。因为它是流式的所以这个方法可以被调用多次。

#### sign.sign(private_key[, output_format])#

根据所有通过`update`方法传入的数据计算数字签名。

`private_key`可以是一个对象或一个字符串，如果`private_key`是一个字符串，那么它被当做没有密码的密钥。

__private_key__:
 - key : 包含`PEM`编码私钥的字符串。
 - passphrase : 一个私钥密码的字符串。
 
返回的数字签名编码由`output_format`决定，可以是`'binary'`，`'hex'`或`'base64'`。如果没有指定编码，会返回一个`buffer`。

注意，在调用了`sign()`后，`sign`对象不能再使用了。

#### crypto.createVerify(algorithm)#

使用给定的算法，创建并返回一个验证器对象。这个对象是`sign`对象的镜像。

#### Class: Verify#

用来验证数字签名的类。

由`crypto.createVerify`返回。

`Verify`对象是一个可写流。写入的数据用来验证提供的数字签名。一旦所有的数据被写入，`verify`方法会返回`true`如果提供的数字签名有效。遗留的`update`方法也支持。

#### verifier.update(data)#

使用`data`更新`verifier `对象。因为它是流式的所以这个方法可以被调用多次。

#### verifier.verify(object, signature[, signature_format])#

通过使用`object`和`signature`验证被签名的数据。`object`是一个包含了PEM编码对象的字符串，这个对象可以是RSA公钥，DSA公钥或X.509证书。`signature `是先前计算出来的数字签名，`signature_format`可以是`'binary'`，`'hex'`或`'base64'`。如果没有指定编码，那么希望收到一个`buffer`。

返回值是`true`或`false`根据数字签名对于数据和公钥的有效性。

注意，在调用了`verify()`后，`verifier`对象不能再使用了。

#### crypto.createDiffieHellman(prime_length[, generator])#

创建一个迪菲－赫尔曼密钥交换对象（Diffie-Hellman key exchange object），并且根据`prime_length`生成一个质数，可以指定一个可选的数字生成器。如果没有指定生成器，将使用`2`。

#### crypto.createDiffieHellman(prime[, prime_encoding][, generator][, generator_encoding])#

通过给定的质数，和可选的生成器，创建一个迪菲－赫尔曼密钥交换对象（Diffie-Hellman key exchange object）。`generator`可以是一个数字，字符串或`Buffer`。如果没有指定生成器，将使用`2`。`prime_encoding`和`generator_encoding`可以是`'binary'`，`'hex'`或`'base64'`。如果没有指定`prime_encoding`，那么希望`prime`是一个`Buffer`。如果没有指定`generator_encoding`，那么希望`generator`是一个`Buffer`。

#### Class: DiffieHellman#

叫来创建迪菲－赫尔曼密钥交换的类。

通过`crypto.createDiffieHellman`返回。

#### diffieHellman.verifyError#

一个包含了所有警告和/或错误的位域，作为检查初始化时的执行结果。以下是这个属性的合法属性（被常量模块定义）：

 - DH_CHECK_P_NOT_SAFE_PRIME
 - DH_CHECK_P_NOT_PRIME
 - DH_UNABLE_TO_CHECK_GENERATOR
 - DH_NOT_SUITABLE_GENERATOR

#### diffieHellman.generateKeys([encoding])#

生成一个 私和公 迪菲－赫尔曼 密钥值，并且返回一个指定编码的公钥。这个密钥可以被转移给第三方。编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### diffieHellman.computeSecret(other_public_key[, input_encoding][, output_encoding])#

使用`other_public_key`作为第三方密钥来计算共享秘密（shared secret），并且返回计算结果。提供的密钥会以`input_encoding`来解读，并且秘密以`output_encoding`来编码。编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

如果没有指定`output_encoding`，那么会返回一个`buffer`。

#### diffieHellman.getPrime([encoding])#

根据指定编码返回一个迪菲－赫尔曼质数，编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### diffieHellman.getGenerator([encoding])#

根据指定编码返回一个迪菲－赫尔曼生成器，编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### diffieHellman.getPublicKey([encoding])#

根据指定编码返回一个迪菲－赫尔曼公钥，编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### diffieHellman.getPrivateKey([encoding])#

根据指定编码返回一个迪菲－赫尔曼私钥，编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### diffieHellman.setPublicKey(public_key[, encoding])#

设置迪菲－赫尔曼公钥，密钥编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么期望接收一个`buffer`。

#### diffieHellman.setPrivateKey(private_key[, encoding])#

设置迪菲－赫尔曼私钥，密钥编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么期望接收一个`buffer`。

#### crypto.getDiffieHellman(group_name)#

创建一个预定义的迪菲－赫尔曼密钥交换对象。支持的群组有：'modp1', 'modp2', 'modp5' (由RFC 2412定义) 和 'modp14', 'modp15', 'modp16', 'modp17', 'modp18' (由RFC 3526定义)。返回的对象模仿`crypto.createDiffieHellman()`创建的对象的借口，但是不允许交换密钥（如通过`diffieHellman.setPublicKey()`）。执行这套流程的好处是双方不需要事先生成或交换组余数，节省了处理和通信时间。

例子（获取一个共享秘密）：

```js
var crypto = require('crypto');
var alice = crypto.getDiffieHellman('modp5');
var bob = crypto.getDiffieHellman('modp5');

alice.generateKeys();
bob.generateKeys();

var alice_secret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
var bob_secret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* alice_secret and bob_secret should be the same */
console.log(alice_secret == bob_secret);
```

#### crypto.createECDH(curve_name)#

使用由`curve_name`指定的预定义椭圆，创建一个椭圆曲线（EC）迪菲－赫尔曼密钥交换对象。使用`getCurves()`来获取可用的椭圆名列表。在最近的发行版中，`openssl ecparam -list_curves`命令也会展示可用的椭圆曲线的名字和简述。

#### Class: ECDH#

用于EC迪菲－赫尔曼密钥交换的类。

由`crypto.createECDH`返回。

#### ECDH.generateKeys([encoding[, format]])#

生成一个 私/公 EC迪菲－赫尔曼密钥值，并且返回指定格式和编码的公钥。这个密钥可以被转移给第三方。

`format`指定点的编码，可以是`'compressed'`，`'uncompressed'`或`'hybrid'`。如果没有指定，那么点将是`'uncompressed'`格式。

编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### ECDH.computeSecret(other_public_key[, input_encoding][, output_encoding])#

使用`other_public_key`作为第三方密钥来计算共享秘密（shared secret），并且返回计算结果。提供的密钥会以`input_encoding`来解读，并且秘密以`output_encoding`来编码。编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

如果没有指定`output_encoding`，那么会返回一个`buffer`。

#### ECDH.getPublicKey([encoding[, format]])#

返回指定编码和格式的EC迪菲－赫尔曼公钥。

`format`指定点的编码，可以是`'compressed'`，`'uncompressed'`或`'hybrid'`。如果没有指定，那么点将是`'uncompressed'`格式。

编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### ECDH.getPrivateKey([encoding])#

返回指定编码的EC迪菲－赫尔曼私钥，编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么会返回一个`buffer`。

#### ECDH.setPublicKey(public_key[, encoding])#

设置EC迪菲－赫尔曼公钥。密钥编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么期望接收一个`buffer`。

#### ECDH.setPrivateKey(private_key[, encoding])#

设置EC迪菲－赫尔曼私钥。密钥编码可以是`'binary'`，`'hex'`或`'base64'`。如果没有提供编码，那么期望接收一个`buffer`。

例子（获取一个共享秘密）：

```js
var crypto = require('crypto');
var alice = crypto.createECDH('secp256k1');
var bob = crypto.createECDH('secp256k1');

alice.generateKeys();
bob.generateKeys();

var alice_secret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
var bob_secret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* alice_secret and bob_secret should be the same */
console.log(alice_secret == bob_secret);
```

#### crypto.pbkdf2(password, salt, iterations, keylen[, digest], callback)#

异步PBKDF2函数。提供被选择的HAMC摘要函数（默认为SHA1）来获取一个请求长度的密码密钥，盐和迭代数。回调函数有两个参数：（`err`，`derivedKey`）。

例子：

```js
crypto.pbkdf2('secret', 'salt', 4096, 512, 'sha256', function(err, key) {
  if (err)
    throw err;
  console.log(key.toString('hex'));  // 'c5e478d...1469e50'
});
```

可用通过`crypto.getHashes()`获取支持的摘要函数列表。

#### crypto.pbkdf2Sync(password, salt, iterations, keylen[, digest])

同步PBKDF2函数。返回`derivedKey`或抛出错误。

#### crypto.randomBytes(size[, callback])#

生成有密码图谱一般健壮的伪随机数据，用处：

```js
// async
crypto.randomBytes(256, function(ex, buf) {
  if (ex) throw ex;
  console.log('Have %d bytes of random data: %s', buf.length, buf);
});

// sync
try {
  var buf = crypto.randomBytes(256);
  console.log('Have %d bytes of random data: %s', buf.length, buf);
} catch (ex) {
  // handle error
  // most likely, entropy sources are drained
}
```

注意：如果熵不足，那么它会阻塞。尽管它从不话费超过几毫秒。唯一可以想到的阻塞是情况是，当整个系统的熵还是很低时，在其之后启动。

#### Class: Certificate#

这个类用来处理已签名公钥 & 挑战（challenges）。最常用的是它的一系列处理`<keygen>`元素的函数。`http://www.openssl.org/docs/apps/spkac.html`。

通过`crypto.Certificate`返回。

#### Certificate.verifySpkac(spkac)#

返回`ture`或`false`，依赖于SPKAC的有效性。

#### Certificate.exportChallenge(spkac)#

导出编码好的公钥从指定的SPKAC。

#### Certificate.exportPublicKey(spkac)#

导出编码好的挑战（challenge）从指定的SPKAC。

#### crypto.publicEncrypt(public_key, buffer)#

使用`public_key`加密`buffer`。目前只支持RSA。

`public_key`可是是一个对象或一个字符串。如果`public_key`是一个字符串，它会被视作没有密码的密钥并且将使用`RSA_PKCS1_OAEP_PADDING`。因为`RSA`公钥可以用来从你传递给这个方法的密钥来获取。

__public_key__:
 - key : 一个包含PEM加密的私钥字符串
 - passphrase : 一个可选的私钥密码字符串
 - __padding__ : 一个可选的填充值，以下值之一：
  - constants.RSA_NO_PADDING
  - constants.RSA_PKCS1_PADDING
  - constants.RSA_PKCS1_OAEP_PADDING

注意：所有的填充值都被常量模块所定义。

#### crypto.publicDecrypt(public_key, buffer)#

详情参阅上文。与`crypto.publicEncrypt`有相同API。默认填充值是`RSA_PKCS1_PADDING`。

#### crypto.privateDecrypt(private_key, buffer)#

使用`private_key`解密`buffer`。

`private_key`可以是一个对象或一个字符串。如果`private_key`是一个字符串，它会当做没有密码的密钥，并且使用`RSA_PKCS1_OAEP_PADDING`。

__public_key__:
 - key : 一个包含PEM加密的私钥字符串
 - passphrase : 一个可选的私钥密码字符串
 - __padding__ : 一个可选的填充值，以下值之一：
  - constants.RSA_NO_PADDING
  - constants.RSA_PKCS1_PADDING
  - constants.RSA_PKCS1_OAEP_PADDING

注意：所有的填充值都被常量模块所定义。

#### crypto.privateEncrypt(private_key, buffer)#

详情参阅上文。与`crypto.privateDecrypt`有相同API。默认填充值是`RSA_PKCS1_PADDING`。

#### crypto.DEFAULT_ENCODING#

默认编码是用于接受字符串或`buffer`的函数。默认值是`'buffer'`，所以默认是使用`Buffer`对象的。这被用来与旧的以`'binary'`为默认编码的程序更好地兼容。

注意新的程序仍可能期望使用`buffer`，所以只将它作为一个临时措施。

#### 近期的API改变

`Crypto`模块在还没有统一的流API概念，以及没有`Buffer`对象来处理二进制数据前就加入了`Node.js`。

因为这样，它的流类没有其他`io.js`类的典型类，而且很多方法默认接受和返回二进制字符串而不是`Buffer`。这些函数将被改成默认接受和返回`Buffer`。

这对于一些但不是所有的使用场景来说是巨大的改变。

例如，如果你现在对`Sign`类使用默认参数，并且传递`Verify`类的结果，不检查数据，那么在以前它将会继续工作。在你曾经得到二进制字符串的地方，你将会得到一个`Buffer`。

但是，如果你正在使用那些使用字符串可以，但使用`Buffer`不能工作的数据（如连接它们，存储进数据库等）。或者对`crypto`函数不传递编码参数来传递二进制字符串。那么以后，你需要提供你想要指定的编码。如果要将默认的使用风格，转换为旧风格的话，将`crypto.DEFAULT_ENCODING`域设置为`'binary'`。注意新的程序仍可能期望接受`buffer`，所以这仅作为一个临时措施。