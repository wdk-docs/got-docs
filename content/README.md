# Got 文档

> 人性化和强大的 Node.js HTTP 请求库

<!-- [![Coverage Status](https://codecov.io/gh/sindresorhus/got/branch/main/graph/badge.svg)](https://codecov.io/gh/sindresorhus/got/branch/main) -->

[![Downloads](https://img.shields.io/npm/dm/got.svg)](https://npmjs.com/got)
[![Install size](https://packagephobia.com/badge?p=got)](https://packagephobia.com/result?p=got)

[了解 Got 与其他 HTTP 库的比较](#comparison)
  
---  

对于浏览器使用，我们推荐同样的人[Ky](https://github.com/sindresorhus/ky)。

---      
   
**支持问题应该问[这里](https://github.com/sindresorhus/got/discussions).**

## 安装 

```sh
npm install got
```

!!! Warning

    这个包是原生的[ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)，不再提供CommonJS导出。
    如果你的项目使用CommonJS，你将不得不[转换为ESM](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c)或使用[dynamic  `import()` ](https://v8.dev/features/dynamic-import)函数。
    请不要打开关于CommonJS / ESM的问题。
    我们只会将关键的安全问题移植到Got v11，而不是功能或错误修复。

## 看一眼

**[快速入门](quick-start.md)指南可用。**

### JSON mode

Got 有一个专门的选项来处理 JSON 有效负载
此外，promise 暴露了一个`.json<T>()`函数，该函数返回 `Promise<T>`。

```js
import got from "got";

const { data } = await got
  .post("https://httpbin.org/anything", {
    json: {
      hello: "world",
    },
  })
  .json();

console.log(data);
//=> {"hello": "world"}
```

对于高级 JSON 用法，请查看[ `parseJson` ](2-options.md# parseJson)和[ `stringifyJson` ](2-options.md# stringifyJson)选项。

**要了解更多有用的技巧，请访问[tips](tips.md)页面。**

## Highlights

- [用于 8K+包和 4M+回购](https://github.com/sindresorhus/got/network/dependents)
- [积极维护](https://github.com/sindresorhus/got/graphs/contributors)
- [深受众多公司信赖](#widely-used)

## 文档

默认情况下，Got 将在失败时重试。要禁用此选项，请将[ `options.retry.limit` ](7-retry.md#retry)设置为 0。

#### 主要 API

- [x] [Promise API](1-promise.md)
- [x] [Options](2-options.md)
- [x] [Stream API](3-streams.md)
- [x] [Pagination API](4-pagination.md)
- [x] [Advanced HTTPS API](5-https.md)
- [x] [HTTP/2 support](2-options.md#http2)
- [x] [`Response` class](3-streams.md#response-2)

#### 超时并重试

- [x] [Advanced timeout handling](6-timeout.md)
- [x] [Retries on failure](7-retry.md)
- [x] [Errors with metadata](8-errors.md)

#### 先进的创造

- [x] [Hooks](9-hooks.md)
- [x] [Instances](10-instances.md)
- [x] [Progress events & other events](3-streams.md#events)
- [x] [Plugins](lets-make-a-plugin.md)
- [x] [Compose](examples/advanced-creation.js)

#### 缓存，代理和 UNIX socket

- [x] [RFC compliant caching](cache.md)
- [x] [Proxy support](tips.md#proxying)
- [x] [Unix Domain Sockets](2-options.md#enableunixsockets)

#### 集成

- [x] [TypeScript support](typescript.md)
- [x] [AWS](tips.md#aws)
- [x] [Testing](tips.md#testing)

---

### 迁移向导

- [Request迁移指南](migration-guides/request.md)
    - [_(注意，Request是不维护的)_](https://github.com/request/request/issues/3142)
- [Axios](migration-guides/axios.md)
- [Node.js](migration-guides/nodejs.md)

## Got 插件

- [`got4aws`](https://github.com/SamVerschueren/got4aws) - Got 方便的包装器与AWS v4签名api交互
- [`gh-got`](https://github.com/sindresorhus/gh-got) - Got 方便包装器与GitHub API交互
- [`gl-got`](https://github.com/singapore/gl-got) - Got 方便包装器与GitLab API交互
- [`gotql`](https://github.com/khaosdoctor/gotql) - Got 方便的包装器使用json解析的查询而不是字符串与GraphQL交互
- [`got-fetch`](https://github.com/alexghr/got-fetch) - `Got` 使用[' fetch '](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)接口
- [`got-scraping`](https://github.com/apify/got-scraping) - Got 专门为网页抓取目的设计的包装器
- [`got-ssrf`](https://github.com/JaneJeon/got-ssrf) - Got 包装器保护服务器端请求免受SSRF攻击

### 遗产

- [travis-got](https://github.com/samverschueren/travis-got) - Got 方便包装与特拉维斯 API 交互
- [graphql-got](https://github.com/kevva/graphql-got) - Got 方便的包装与 GraphQL 交互

## 比较

|                       |        `got`        |  [`node-fetch`][n0]  |        [`ky`][k0]        |   [`axios`][a0]    |   [`superagent`][s0]   |
| --------------------- | :-----------------: | :------------------: | :----------------------: | :----------------: | :--------------------: |
| HTTP/2 support        | :heavy_check_mark:¹ |         :x:          |           :x:            |        :x:         | :heavy_check_mark:\*\* |
| Browser support       |         :x:         | :heavy_check_mark:\* |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Promise API           | :heavy_check_mark:  |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Stream API            | :heavy_check_mark:  |     Node.js only     |           :x:            |        :x:         |   :heavy_check_mark:   |
| Pagination API        | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |          :x:           |
| Request cancelation   | :heavy_check_mark:  |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| RFC compliant caching | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |          :x:           |
| Cookies (out-of-box)  | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |          :x:           |
| Follows redirects     | :heavy_check_mark:  |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Retries on failure    | :heavy_check_mark:  |         :x:          |    :heavy_check_mark:    |        :x:         |   :heavy_check_mark:   |
| Progress events       | :heavy_check_mark:  |         :x:          | :heavy_check_mark:\*\*\* |    Browser only    |   :heavy_check_mark:   |
| Handles gzip/deflate  | :heavy_check_mark:  |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Advanced timeouts     | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |          :x:           |
| Timings               | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |          :x:           |
| Errors with metadata  | :heavy_check_mark:  |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| JSON mode             | :heavy_check_mark:  |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Custom defaults       | :heavy_check_mark:  |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| Composable            | :heavy_check_mark:  |         :x:          |           :x:            |        :x:         |   :heavy_check_mark:   |
| Hooks                 | :heavy_check_mark:  |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| Issues open           |   [![][gio]][g1]    |    [![][nio]][n1]    |      [![][kio]][k1]      |   [![][aio]][a1]   |     [![][sio]][s1]     |
| Issues closed         |   [![][gic]][g2]    |    [![][nic]][n2]    |      [![][kic]][k2]      |   [![][aic]][a2]   |     [![][sic]][s2]     |
| Downloads             |    [![][gd]][g3]    |    [![][nd]][n3]     |      [![][kd]][k3]       |   [![][ad]][a3]    |     [![][sd]][s3]      |
| Coverage              |         TBD         |    [![][nc]][n4]     |      [![][kc]][k4]       |   [![][ac]][a4]    |     [![][sc]][s4]      |
| Build                 |    [![][gb]][g5]    |    [![][nb]][n5]     |      [![][kb]][k5]       |   [![][ab]][a5]    |     [![][sb]][s5]      |
| Bugs                  |   [![][gbg]][g6]    |    [![][nbg]][n6]    |      [![][kbg]][k6]      |   [![][abg]][a6]   |     [![][sbg]][s6]     |
| Dependents            |   [![][gdp]][g7]    |    [![][ndp]][n7]    |      [![][kdp]][k7]      |   [![][adp]][a7]   |     [![][sdp]][s7]     |
| Install size          |   [![][gis]][g8]    |    [![][nis]][n8]    |      [![][kis]][k8]      |   [![][ais]][a8]   |     [![][sis]][s8]     |
| GitHub stars          |    [![][gs]][g9]    |    [![][ns]][n9]     |      [![][ks]][k9]       |   [![][as]][a9]    |     [![][ss]][s9]      |
| TypeScript support    |   [![][gts]][g10]   |   [![][nts]][n10]    |     [![][kts]][k10]      |  [![][ats]][a10]   |    [![][sts]][s11]     |
| Last commit           |   [![][glc]][g11]   |   [![][nlc]][n11]    |     [![][klc]][k11]      |  [![][alc]][a11]   |    [![][slc]][s11]     |

它几乎与浏览器的`fetch`API兼容。  
需要手动切换协议。不接受PUSH流，不重用HTTP/2会话。  
目前，只支持`DownloadProgress`事件，不支持`UploadProgress`事件。 
¹ 要求Node.js 15.10.0或以上版本。  
:sparkle: 几乎稳定的特性，但是API可能会改变。不要犹豫，试一试吧!  
:grey_question: 在早期开发阶段的功能。实验。  

<!-- GITHUB -->

[k0]: https://github.com/sindresorhus/ky
[n0]: https://github.com/node-fetch/node-fetch
[a0]: https://github.com/axios/axios
[s0]: https://github.com/visionmedia/superagent

<!-- ISSUES OPEN -->

[gio]: https://img.shields.io/github/issues-raw/sindresorhus/got?color=gray&label
[kio]: https://img.shields.io/github/issues-raw/sindresorhus/ky?color=gray&label
[nio]: https://img.shields.io/github/issues-raw/bitinn/node-fetch?color=gray&label
[aio]: https://img.shields.io/github/issues-raw/axios/axios?color=gray&label
[sio]: https://img.shields.io/github/issues-raw/visionmedia/superagent?color=gray&label
[g1]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[k1]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[n1]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[a1]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[s1]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc

<!-- ISSUES CLOSED -->

[gic]: https://img.shields.io/github/issues-closed-raw/sindresorhus/got?color=blue&label
[kic]: https://img.shields.io/github/issues-closed-raw/sindresorhus/ky?color=blue&label
[nic]: https://img.shields.io/github/issues-closed-raw/bitinn/node-fetch?color=blue&label
[aic]: https://img.shields.io/github/issues-closed-raw/axios/axios?color=blue&label
[sic]: https://img.shields.io/github/issues-closed-raw/visionmedia/superagent?color=blue&label
[g2]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[k2]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[n2]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[a2]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[s2]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc

<!-- DOWNLOADS -->

[gd]: https://img.shields.io/npm/dm/got?color=darkgreen&label
[kd]: https://img.shields.io/npm/dm/ky?color=darkgreen&label
[nd]: https://img.shields.io/npm/dm/node-fetch?color=darkgreen&label
[ad]: https://img.shields.io/npm/dm/axios?color=darkgreen&label
[sd]: https://img.shields.io/npm/dm/superagent?color=darkgreen&label
[g3]: https://www.npmjs.com/package/got
[k3]: https://www.npmjs.com/package/ky
[n3]: https://www.npmjs.com/package/node-fetch
[a3]: https://www.npmjs.com/package/axios
[s3]: https://www.npmjs.com/package/superagent

<!-- COVERAGE -->

[gc]: https://img.shields.io/coveralls/github/sindresorhus/got?color=0b9062&label
[kc]: https://img.shields.io/codecov/c/github/sindresorhus/ky?color=0b9062&label
[nc]: https://img.shields.io/coveralls/github/bitinn/node-fetch?color=0b9062&label
[ac]: https://img.shields.io/coveralls/github/mzabriskie/axios?color=0b9062&label
[sc]: https://img.shields.io/codecov/c/github/visionmedia/superagent?color=0b9062&label
[g4]: https://coveralls.io/github/sindresorhus/got
[k4]: https://codecov.io/gh/sindresorhus/ky
[n4]: https://coveralls.io/github/bitinn/node-fetch
[a4]: https://coveralls.io/github/mzabriskie/axios
[s4]: https://codecov.io/gh/visionmedia/superagent

<!-- BUILD -->

[gb]: https://github.com/sindresorhus/got/actions/workflows/main.yml/badge.svg
[kb]: https://github.com/sindresorhus/ky/actions/workflows/main.yml/badge.svg
[nb]: https://img.shields.io/travis/bitinn/node-fetch?label
[ab]: https://img.shields.io/travis/axios/axios?label
[sb]: https://img.shields.io/travis/visionmedia/superagent?label
[g5]: https://github.com/sindresorhus/got/actions/workflows/main.yml
[k5]: https://github.com/sindresorhus/ky/actions/workflows/main.yml
[n5]: https://travis-ci.org/github/bitinn/node-fetch
[a5]: https://travis-ci.org/github/axios/axios
[s5]: https://travis-ci.org/github/visionmedia/superagent

<!-- BUGS -->

[gbg]: https://img.shields.io/github/issues-raw/sindresorhus/got/bug?color=darkred&label
[kbg]: https://img.shields.io/github/issues-raw/sindresorhus/ky/bug?color=darkred&label
[nbg]: https://img.shields.io/github/issues-raw/bitinn/node-fetch/bug?color=darkred&label
[abg]: https://img.shields.io/github/issues-raw/axios/axios/type:confirmed%20bug?color=darkred&label
[sbg]: https://img.shields.io/github/issues-raw/visionmedia/superagent/Bug?color=darkred&label
[g6]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[k6]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[n6]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[a6]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3A%22type%3Aconfirmed+bug%22
[s6]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3ABug

<!-- DEPENDENTS -->

[gdp]: https://badgen.net/npm/dependents/got?color=orange&label
[kdp]: https://badgen.net/npm/dependents/ky?color=orange&label
[ndp]: https://badgen.net/npm/dependents/node-fetch?color=orange&label
[adp]: https://badgen.net/npm/dependents/axios?color=orange&label
[sdp]: https://badgen.net/npm/dependents/superagent?color=orange&label
[g7]: https://www.npmjs.com/package/got?activeTab=dependents
[k7]: https://www.npmjs.com/package/ky?activeTab=dependents
[n7]: https://www.npmjs.com/package/node-fetch?activeTab=dependents
[a7]: https://www.npmjs.com/package/axios?activeTab=dependents
[s7]: https://www.npmjs.com/package/visionmedia?activeTab=dependents

<!-- INSTALL SIZE -->

[gis]: https://badgen.net/packagephobia/install/got?color=blue&label
[kis]: https://badgen.net/packagephobia/install/ky?color=blue&label
[nis]: https://badgen.net/packagephobia/install/node-fetch?color=blue&label
[ais]: https://badgen.net/packagephobia/install/axios?color=blue&label
[sis]: https://badgen.net/packagephobia/install/superagent?color=blue&label
[g8]: https://packagephobia.com/result?p=got
[k8]: https://packagephobia.com/result?p=ky
[n8]: https://packagephobia.com/result?p=node-fetch
[a8]: https://packagephobia.com/result?p=axios
[s8]: https://packagephobia.com/result?p=superagent

<!-- GITHUB STARS -->

[gs]: https://img.shields.io/github/stars/sindresorhus/got?color=white&label
[ks]: https://img.shields.io/github/stars/sindresorhus/ky?color=white&label
[ns]: https://img.shields.io/github/stars/bitinn/node-fetch?color=white&label
[as]: https://img.shields.io/github/stars/axios/axios?color=white&label
[ss]: https://img.shields.io/github/stars/visionmedia/superagent?color=white&label
[g9]: https://github.com/sindresorhus/got
[k9]: https://github.com/sindresorhus/ky
[n9]: https://github.com/node-fetch/node-fetch
[a9]: https://github.com/axios/axios
[s9]: https://github.com/visionmedia/superagent

<!-- TYPESCRIPT SUPPORT -->

[gts]: https://badgen.net/npm/types/got?label
[kts]: https://badgen.net/npm/types/ky?label
[nts]: https://badgen.net/npm/types/node-fetch?label
[ats]: https://badgen.net/npm/types/axios?label
[sts]: https://badgen.net/npm/types/superagent?label
[g10]: https://github.com/sindresorhus/got
[k10]: https://github.com/sindresorhus/ky
[n10]: https://github.com/node-fetch/node-fetch
[a10]: https://github.com/axios/axios
[s10]: https://github.com/visionmedia/superagent

<!-- LAST COMMIT -->

[glc]: https://img.shields.io/github/last-commit/sindresorhus/got?color=gray&label
[klc]: https://img.shields.io/github/last-commit/sindresorhus/ky?color=gray&label
[nlc]: https://img.shields.io/github/last-commit/bitinn/node-fetch?color=gray&label
[alc]: https://img.shields.io/github/last-commit/axios/axios?color=gray&label
[slc]: https://img.shields.io/github/last-commit/visionmedia/superagent?color=gray&label
[g11]: https://github.com/sindresorhus/got/commits
[k11]: https://github.com/sindresorhus/ky/commits
[n11]: https://github.com/node-fetch/node-fetch/commits
[a11]: https://github.com/axios/axios/commits
[s11]: https://github.com/visionmedia/superagent/commits

[点击这里][installsizeofthedependencies]查看got依赖的安装大小。

[installsizeofthedependencies]: https://packagephobia.com/result?p=@sindresorhus/is@4.0.1,@szmarczak/http-timer@4.0.5,@types/cacheable-request@6.0.1,@types/responselike@1.0.0,cacheable-lookup@6.0.0,cacheable-request@7.0.2,decompress-response@6.0.0,get-stream@6.0.1,http2-wrapper@2.0.5,lowercase-keys@2.0.0,p-cancelable@2.1.1,responselike@2.0.0

## 维护人员

| [![Sindre Sorhus](https://github.com/sindresorhus.png?size=100)](https://sindresorhus.com) | [![Szymon Marczak](https://github.com/szmarczak.png?size=100)](https://github.com/szmarczak) |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| [Sindre Sorhus](https://sindresorhus.com)                                                  | [Szymon Marczak](https://github.com/szmarczak)                                               |

###### 前

- [Vsevolod Strukchinsky](https://github.com/floatdrop)
- [Alexander Tesfamichael](https://github.com/alextes)
- [Brandon Smith](https://github.com/brandon93s)
- [Luke Childs](https://github.com/lukechilds)
- [Giovanni Minotti](https://github.com/Giotino)

<a name="widely-used"></a>

## 这些了不起的公司正在使用 Got

<table>
<tbody>
	<tr>
		<td align="center">
			<a href="https://segment.com">
				<img width="90" valign="middle" src="https://user-images.githubusercontent.com/697676/47693700-ddb62500-dbb7-11e8-8332-716a91010c2d.png">
			</a>
		</td>
		<td align="center">
			<a href="https://antora.org">
				<img width="100" valign="middle" src="https://user-images.githubusercontent.com/79351/47706840-d874cc80-dbef-11e8-87c6-5f0c60cbf5dc.png">
			</a>
		</td>
		<td align="center">
			<a href="https://getvoip.com">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/10832620/47869404-429e9480-dddd-11e8-8a7a-ca43d7f06020.png">
			</a>
		</td>
		<td align="center">
			<a href="https://github.com/exoframejs/exoframe">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/365944/47791460-11a95b80-dd1a-11e8-9070-e8f2a215e03a.png">
			</a>
		</td>
	</tr>
	<tr>
		<td align="center">
			<a href="http://karaokes.moe">
				<img width="140" valign="middle" src="https://camo.githubusercontent.com/6860e5fa4684c14d8e1aa65df0aba4e6808ea1a9/687474703a2f2f6b6172616f6b65732e6d6f652f6173736574732f696d616765732f696e6465782e706e67">
			</a>
		</td>
		<td align="center">
			<a href="https://github.com/renovatebot/renovate">
				<img width="150" valign="middle" src="https://camo.githubusercontent.com/206d470ac709b9a702a97b0c08d6f389a086793d/68747470733a2f2f72656e6f76617465626f742e636f6d2f696d616765732f6c6f676f2e737667">
			</a>
		</td>
		<td align="center">
			<a href="https://resist.bot">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/3322287/51992724-28736180-2473-11e9-9764-599cfda4b012.png">
			</a>
		</td>
		<td align="center">
			<a href="https://www.naturalcycles.com">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/170270/92244143-d0a8a200-eec2-11ea-9fc0-1c07f90b2113.png">
			</a>
		</td>
	</tr>
	<tr>
		<td align="center">
			<a href="https://microlink.io">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/36894700/91992974-1cc5dc00-ed35-11ea-9d04-f58b42ce6a5e.png">
			</a>
		</td>
		<td align="center">
			<a href="https://radity.com">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/29518613/91814036-97fb9500-ec44-11ea-8c6c-d198cc23ca29.png">
			</a>
		</td>
		<td align="center">
			<a href="https://apify.com">
				<img width="150" valign="middle" src="https://user-images.githubusercontent.com/23726914/128673143-958b5930-b677-40ef-8087-5698a0c29c45.png">
			</a>
		</td>
	</tr>
</tbody>
</table>

<!-- <br> -->

<!-- *Creating an awesome product? Open an issue to get listed here.* -->

<br>

> 段是一个快乐的用户!Got 为应用程序对话的主后端 API 提供了动力。它被我们的内部 RPC 客户端使用，我们用它来与所有微服务通信。
>
> — <a href="https://github.com/vadimdemedes">Vadim Demedes</a>

> Antora 是一个用于创建文档站点的静态站点生成器，它使用 Got 来下载 UI 包。
> 在 Antora 中，UI 包(又名主题)是作为一个单独的项目维护的。
> 该项目将 UI 导出为 zip 文件，我们称之为 UI 包。
> 主站点生成器使用 Got 从 URL 下载该 UI，并将其传输到 vinyl-zip 以提取文件。
> 这些文件将继续用于创建 HTML 页面和支持资产。
>
> — <a href="https://github.com/mojavelinux">Dan Allen</a>

> GetVoIP在生产中愉快地使用了Got。Got的独特功能之一是处理Unix套接字的能力，这使我们能够为我们的docker堆栈构建一个完整的控制接口。
>
> — <a href="https://github.com/danielkalen">Daniel Kalen</a>

> 我们在exframe内部使用get来处理CLI和服务器之间的所有通信。Exoframe是一个自托管工具，允许使用Docker进行简单的单命令部署。
>
> — <a href="https://github.com/yamalight">Tim Ermilov</a>

> 卡拉ok Mugen使用Got从其在线服务器获取内容更新。
>
> — <a href="https://github.com/AxelTerizaki">Axel Terizaki</a>

> Renovate使用Got、gh-got和gl-got每天向GitHub、GitLab、npmjs、PyPi、Packagist、Docker Hub、Terraform、CircleCI等发送数百万个查询。
>
> — <a href="https://github.com/rarkins">Rhys Arkins</a>

> Resistbot使用Got从API前端进行通信，其中所有通信都进入到后台的官方查找数据库。
>
> — <a href="https://github.com/chris-erickson">Chris Erickson</a>

> Natural Cycles使用Got与各种第三方REST api(超过9000个!)进行通信。
>
> — <a href="https://github.com/kirillgroshkov">Kirill Groshkov</a>

> Microlink是一个云浏览器作为API服务，广泛使用get作为主要的HTTP客户端，每月处理约2200万个请求，每次需要执行网络调用。
>
> — <a href="https://github.com/Kikobeats">Kiko Beats</a>

> 我们在Radity使用Got。谢谢你这么了不起的工作!
>
> — <a href="https://github.com/MirzayevFarid">Mirzayev Farid</a>

> 多年来，Got一直是Apify数据抓取的关键组成部分。我们每个月都用它从数十亿个网页中提取数据，我们真的很欣赏它强大的API和可扩展性，它使我们能够在Got的基础上构建我们自己的专用HTTP客户端。支持也一直是一流的。
>
> — <a href="https://github.com/mnmkng">Ondra Urban</a>

## 为企业

可作为Tidelift订阅的一部分。

`got`和其他数千个软件包的维护者正在与Tidelift合作，为您构建应用程序时使用的开源依赖项提供商业支持和维护。节省时间、降低风险并改善代码运行状况，同时向您所使用的依赖项的维护者支付费用。[了解更多](https://tidelift.com/subscription/pkg/npm-got?utm_source=npm-got&utm_medium=referral&utm_campaign=enterprise&utm_term=repo)