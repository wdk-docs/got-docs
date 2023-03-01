---
title: "与其它项目对照"
linkTitle: "项目对照"
weight: 3
type: "docs"
---

|                       |       `got`        |  [`request`][r0]   |  [`node-fetch`][n0]  |        [`ky`][k0]        |   [`axios`][a0]    |   [`superagent`][s0]   |
| --------------------- | :----------------: | :----------------: | :------------------: | :----------------------: | :----------------: | :--------------------: |
| HTTP/2 support        |  :grey_question:   |        :x:         |         :x:          |           :x:            |        :x:         | :heavy_check_mark:\*\* |
| Browser support       |        :x:         |        :x:         | :heavy_check_mark:\* |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Promise API           | :heavy_check_mark: | :heavy_check_mark: |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Stream API            | :heavy_check_mark: | :heavy_check_mark: |     Node.js only     |           :x:            |        :x:         |   :heavy_check_mark:   |
| Pagination API        |     :sparkle:      |        :x:         |         :x:          |           :x:            |        :x:         |          :x:           |
| Request cancelation   | :heavy_check_mark: |        :x:         |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| RFC compliant caching | :heavy_check_mark: |        :x:         |         :x:          |           :x:            |        :x:         |          :x:           |
| Cookies (out-of-box)  | :heavy_check_mark: | :heavy_check_mark: |         :x:          |           :x:            |        :x:         |          :x:           |
| Follows redirects     | :heavy_check_mark: | :heavy_check_mark: |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Retries on failure    | :heavy_check_mark: |        :x:         |         :x:          |    :heavy_check_mark:    |        :x:         |   :heavy_check_mark:   |
| Progress events       | :heavy_check_mark: |        :x:         |         :x:          | :heavy_check_mark:\*\*\* |    Browser only    |   :heavy_check_mark:   |
| Handles gzip/deflate  | :heavy_check_mark: | :heavy_check_mark: |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Advanced timeouts     | :heavy_check_mark: |        :x:         |         :x:          |           :x:            |        :x:         |          :x:           |
| Timings               | :heavy_check_mark: | :heavy_check_mark: |         :x:          |           :x:            |        :x:         |          :x:           |
| Errors with metadata  | :heavy_check_mark: |        :x:         |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| JSON mode             | :heavy_check_mark: | :heavy_check_mark: |  :heavy_check_mark:  |    :heavy_check_mark:    | :heavy_check_mark: |   :heavy_check_mark:   |
| Custom defaults       | :heavy_check_mark: | :heavy_check_mark: |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| Composable            | :heavy_check_mark: |        :x:         |         :x:          |           :x:            |        :x:         |   :heavy_check_mark:   |
| Hooks                 | :heavy_check_mark: |        :x:         |         :x:          |    :heavy_check_mark:    | :heavy_check_mark: |          :x:           |
| Issues open           |   [![][gio]][g1]   |   [![][rio]][r1]   |    [![][nio]][n1]    |      [![][kio]][k1]      |   [![][aio]][a1]   |     [![][sio]][s1]     |
| Issues closed         |   [![][gic]][g2]   |   [![][ric]][r2]   |    [![][nic]][n2]    |      [![][kic]][k2]      |   [![][aic]][a2]   |     [![][sic]][s2]     |
| Downloads             |   [![][gd]][g3]    |   [![][rd]][r3]    |    [![][nd]][n3]     |      [![][kd]][k3]       |   [![][ad]][a3]    |     [![][sd]][s3]      |
| Coverage              |   [![][gc]][g4]    |   [![][rc]][r4]    |    [![][nc]][n4]     |      [![][kc]][k4]       |   [![][ac]][a4]    |     [![][sc]][s4]      |
| Build                 |   [![][gb]][g5]    |   [![][rb]][r5]    |    [![][nb]][n5]     |      [![][kb]][k5]       |   [![][ab]][a5]    |     [![][sb]][s5]      |
| Bugs                  |   [![][gbg]][g6]   |   [![][rbg]][r6]   |    [![][nbg]][n6]    |      [![][kbg]][k6]      |   [![][abg]][a6]   |     [![][sbg]][s6]     |
| Dependents            |   [![][gdp]][g7]   |   [![][rdp]][r7]   |    [![][ndp]][n7]    |      [![][kdp]][k7]      |   [![][adp]][a7]   |     [![][sdp]][s7]     |
| Install size          |   [![][gis]][g8]   |   [![][ris]][r8]   |    [![][nis]][n8]    |      [![][kis]][k8]      |   [![][ais]][a8]   |     [![][sis]][s8]     |

- It's almost API compatible with the browser `fetch` API.\
  - Need to switch the protocol manually. Doesn't accept PUSH streams and doesn't reuse HTTP/2 sessions.\
    - Currently, only `DownloadProgress` event is supported, `UploadProgress` event is not supported.\
      :sparkle: Almost-stable feature, but the API may change. Don't hestitate to try it out!\
      :grey_question: Feature in early stage of development. Very experimental.

<!-- GITHUB -->

[k0]: https://github.com/sindresorhus/ky
[r0]: https://github.com/request/request
[n0]: https://github.com/bitinn/node-fetch
[a0]: https://github.com/axios/axios
[s0]: https://github.com/visionmedia/superagent

<!-- ISSUES OPEN -->

[gio]: https://badgen.net/github/open-issues/sindresorhus/got?label
[kio]: https://badgen.net/github/open-issues/sindresorhus/ky?label
[rio]: https://badgen.net/github/open-issues/request/request?label
[nio]: https://badgen.net/github/open-issues/bitinn/node-fetch?label
[aio]: https://badgen.net/github/open-issues/axios/axios?label
[sio]: https://badgen.net/github/open-issues/visionmedia/superagent?label
[g1]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[k1]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[r1]: https://github.com/request/request/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[n1]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[a1]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc
[s1]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc

<!-- ISSUES CLOSED -->

[gic]: https://badgen.net/github/closed-issues/sindresorhus/got?label
[kic]: https://badgen.net/github/closed-issues/sindresorhus/ky?label
[ric]: https://badgen.net/github/closed-issues/request/request?label
[nic]: https://badgen.net/github/closed-issues/bitinn/node-fetch?label
[aic]: https://badgen.net/github/closed-issues/axios/axios?label
[sic]: https://badgen.net/github/closed-issues/visionmedia/superagent?label
[g2]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[k2]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[r2]: https://github.com/request/request/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[n2]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[a2]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc
[s2]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aclosed+sort%3Aupdated-desc

<!-- DOWNLOADS -->

[gd]: https://badgen.net/npm/dm/got?label
[kd]: https://badgen.net/npm/dm/ky?label
[rd]: https://badgen.net/npm/dm/request?label
[nd]: https://badgen.net/npm/dm/node-fetch?label
[ad]: https://badgen.net/npm/dm/axios?label
[sd]: https://badgen.net/npm/dm/superagent?label
[g3]: https://www.npmjs.com/package/got
[k3]: https://www.npmjs.com/package/ky
[r3]: https://www.npmjs.com/package/request
[n3]: https://www.npmjs.com/package/node-fetch
[a3]: https://www.npmjs.com/package/axios
[s3]: https://www.npmjs.com/package/superagent

<!-- COVERAGE -->

[gc]: https://badgen.net/coveralls/c/github/sindresorhus/got?label
[kc]: https://badgen.net/codecov/c/github/sindresorhus/ky?label
[rc]: https://badgen.net/coveralls/c/github/request/request?label
[nc]: https://badgen.net/coveralls/c/github/bitinn/node-fetch?label
[ac]: https://badgen.net/coveralls/c/github/mzabriskie/axios?label
[sc]: https://badgen.net/codecov/c/github/visionmedia/superagent?label
[g4]: https://coveralls.io/github/sindresorhus/got
[k4]: https://codecov.io/gh/sindresorhus/ky
[r4]: https://coveralls.io/github/request/request
[n4]: https://coveralls.io/github/bitinn/node-fetch
[a4]: https://coveralls.io/github/mzabriskie/axios
[s4]: https://codecov.io/gh/visionmedia/superagent

<!-- BUILD -->

[gb]: https://badgen.net/travis/sindresorhus/got?label
[kb]: https://badgen.net/travis/sindresorhus/ky?label
[rb]: https://badgen.net/travis/request/request?label
[nb]: https://badgen.net/travis/bitinn/node-fetch?label
[ab]: https://badgen.net/travis/axios/axios?label
[sb]: https://badgen.net/travis/visionmedia/superagent?label
[g5]: https://travis-ci.org/sindresorhus/got
[k5]: https://travis-ci.org/sindresorhus/ky
[r5]: https://travis-ci.org/request/request
[n5]: https://travis-ci.org/bitinn/node-fetch
[a5]: https://travis-ci.org/axios/axios
[s5]: https://travis-ci.org/visionmedia/superagent

<!-- BUGS -->

[gbg]: https://badgen.net/github/label-issues/sindresorhus/got/bug/open?label
[kbg]: https://badgen.net/github/label-issues/sindresorhus/ky/bug/open?label
[rbg]: https://badgen.net/github/label-issues/request/request/Needs%20investigation/open?label
[nbg]: https://badgen.net/github/label-issues/bitinn/node-fetch/bug/open?label
[abg]: https://badgen.net/github/label-issues/axios/axios/type:bug/open?label
[sbg]: https://badgen.net/github/label-issues/visionmedia/superagent/Bug/open?label
[g6]: https://github.com/sindresorhus/got/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[k6]: https://github.com/sindresorhus/ky/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[r6]: https://github.com/request/request/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3A"Needs+investigation"
[n6]: https://github.com/bitinn/node-fetch/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Abug
[a6]: https://github.com/axios/axios/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Atype:bug
[s6]: https://github.com/visionmedia/superagent/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3ABug

<!-- DEPENDENTS -->

[gdp]: https://badgen.net/npm/dependents/got?label
[kdp]: https://badgen.net/npm/dependents/ky?label
[rdp]: https://badgen.net/npm/dependents/request?label
[ndp]: https://badgen.net/npm/dependents/node-fetch?label
[adp]: https://badgen.net/npm/dependents/axios?label
[sdp]: https://badgen.net/npm/dependents/superagent?label
[g7]: https://www.npmjs.com/package/got?activeTab=dependents
[k7]: https://www.npmjs.com/package/ky?activeTab=dependents
[r7]: https://www.npmjs.com/package/request?activeTab=dependents
[n7]: https://www.npmjs.com/package/node-fetch?activeTab=dependents
[a7]: https://www.npmjs.com/package/axios?activeTab=dependents
[s7]: https://www.npmjs.com/package/visionmedia?activeTab=dependents

<!-- INSTALL SIZE -->

[gis]: https://badgen.net/packagephobia/install/got?label
[kis]: https://badgen.net/packagephobia/install/ky?label
[ris]: https://badgen.net/packagephobia/install/request?label
[nis]: https://badgen.net/packagephobia/install/node-fetch?label
[ais]: https://badgen.net/packagephobia/install/axios?label
[sis]: https://badgen.net/packagephobia/install/superagent?label
[g8]: https://packagephobia.now.sh/result?p=got
[k8]: https://packagephobia.now.sh/result?p=ky
[r8]: https://packagephobia.now.sh/result?p=request
[n8]: https://packagephobia.now.sh/result?p=node-fetch
[a8]: https://packagephobia.now.sh/result?p=axios
[s8]: https://packagephobia.now.sh/result?p=superagent

[Click here][installsizeofthedependencies] to see the install size of the Got dependencies.

[installsizeofthedependencies]: https://packagephobia.now.sh/result?p=@sindresorhus/is@1.0.0,@szmarczak/http-timer@3.1.0,@types/cacheable-request@6.0.1,cacheable-lookup@0.2.1,cacheable-request@7.0.0,decompress-response@5.0.0,duplexer3@0.1.4,get-stream@5.0.0,lowercase-keys@2.0.0,mimic-response@2.0.0,p-cancelable@2.0.0,responselike@2.0.0,to-readable-stream@2.0.0,type-fest@0.8.0
