# 高级 HTTPS API

## `https`

**类型: `object`**

此选项表示用于发出HTTPS请求的选项。

### `alpnProtocols`

**类型: `string[]`**  
**默认: `['http/1.1']`**

可接受的[ALPN](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)协议。
如果`http2`选项为`true`，则默认为`['h2', 'http/1.1']`。

### `rejectUnauthorized`

**类型: `boolean`**  
**默认: `true`**

如果为`true`，它将抛出无效证书，例如过期或自签名证书。

### `checkServerIdentity`

**类型: `(hostname: string, certificate: DetailedPeerCertificate) => Error | undefined`**  
**默认: `tls.checkServerIdentity`**

证书的自定义检查。用于固定证书。


如果检查成功，函数必须返回`undefined`。
如果失败，应该返回`Error`。

!!! note

    为了调用该函数，证书不能过期、不能自签名，也不能使用不可信的根。

参考[Node.js文档](https://nodejs.org/api/https.html#https_https_request_url_options_callback)。

### `certificateAuthority`

**类型: `string | Buffer | string[] | Buffer[]`**

!!! Note

    该选项已从[`ca` TLS option](https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options)重命名为更好的可读性。

覆盖受信任的[CA](https://en.wikipedia.org/wiki/Certificate_authority)证书。

默认为[Mozilla](https://ccadb-public.secure.force.com/mozilla/IncludedCACertificateReport)提供的CAs.

```js
import got from "got";

// Single Certificate Authority
await got("https://example.com", {
  https: {
    certificateAuthority: fs.readFileSync("./my_ca.pem"),
  },
});
```

### `key`

**类型: `string | Buffer | string[] | Buffer[] | object[]`**

[PEM格式](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail)私钥.

具有不同密码短语的多个密钥可以以`{pem: <string | Buffer>, passphrase: <string>}`数组的形式提供。

!!! note

    加密的密钥将使用`https.passphrase`进行解密。

### `passphrase`

**类型: `string`**

用于单个私钥和/或PFX的共享密码短语。

### `certificate`

**类型: `string | Buffer | string[] | Buffer[]`**

!!! note

    该选项已从[`cert` TLS 选项](https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options)重命名为更好的可读性。

[PEM格式](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail)的证书链[https://en.wikipedia.org/wiki/X.509#Certificate_chains_and_cross-certification]。

每个私钥应该提供一个证书链。

当提供多个证书链时，它们不必与“https.key”中的私钥顺序相同。

### `pfx`

**类型: `string | Buffer | string[] | Buffer[] | object[]`**

[PFX或PKCS12](https://en.wikipedia.org/wiki/PKCS_12)编码的私钥和证书链。
使用`https.pfx`是单独提供`https.key`和`https.certificate`的另一种选择。
PFX通常是加密的，然后使用`https.passphrase`来解密它。

多个PFX可以作为未加密缓冲区数组或对象数组提供，如:

```ts
{
	buffer: string | Buffer,
	passphrase?: string
}
```

### `certificateRevocationLists`

**类型: `string | Buffer | string[] | Buffer[]`**

!!! note

    该选项已从[`crl` TLS 选项](https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options) 重命名为更好的可读性。

## 其他HTTPS选项

[以下选项的文档](https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options)

- `ciphers`
- `dhparam`
- `signatureAlgorithms` (从`sigalgs`重命名)
- `minVersion`
- `maxVersion`
- `honorCipherOrder`
- `tlsSessionLifetime` (从`sessionTimeout`重命名)
- `ecdhCurve`

## 示例

```js
import got from "got";

// 带证书的单密钥
await got("https://example.com", {
  https: {
    key: fs.readFileSync("./client_key.pem"),
    certificate: fs.readFileSync("./client_cert.pem"),
  },
});

// 带有证书的多个密钥(顺序混乱)
await got("https://example.com", {
  https: {
    key: [fs.readFileSync("./client_key1.pem"), fs.readFileSync("./client_key2.pem")],
    certificate: [fs.readFileSync("./client_cert2.pem"), fs.readFileSync("./client_cert1.pem")],
  },
});

// 单密钥与密码短语
await got("https://example.com", {
  https: {
    key: fs.readFileSync("./client_key.pem"),
    certificate: fs.readFileSync("./client_cert.pem"),
    passphrase: "client_key_passphrase",
  },
});

// 具有不同密码的多个密钥
await got("https://example.com", {
  https: {
    key: [
      { pem: fs.readFileSync("./client_key1.pem"), passphrase: "passphrase1" },
      { pem: fs.readFileSync("./client_key2.pem"), passphrase: "passphrase2" },
    ],
    certificate: [fs.readFileSync("./client_cert1.pem"), fs.readFileSync("./client_cert2.pem")],
  },
});

// 带密码短语的单个加密PFX
await got("https://example.com", {
  https: {
    pfx: fs.readFileSync("./fake.pfx"),
    passphrase: "passphrase",
  },
});

// 使用不同密码的多个加密PFX
await got("https://example.com", {
  https: {
    pfx: [
      {
        buffer: fs.readFileSync("./key1.pfx"),
        passphrase: "passphrase1",
      },
      {
        buffer: fs.readFileSync("./key2.pfx"),
        passphrase: "passphrase2",
      },
    ],
  },
});

// 多个加密PFX的单一密码短语
await got("https://example.com", {
  https: {
    passphrase: "passphrase",
    pfx: [
      {
        buffer: fs.readFileSync("./key1.pfx"),
      },
      {
        buffer: fs.readFileSync("./key2.pfx"),
      },
    ],
  },
});
```
