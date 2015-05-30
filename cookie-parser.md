# cookie-parser

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build Status][travis-image]][travis-url]
[![Test Coverage][coveralls-image]][coveralls-url]

解析`Cookie`数据和通过cookie名用对象键值给`req.cookies`赋值。
你可以启用cookie签署支持通过一个`secret`字符串,分配`req.secret`所以它可能使用的其他中间件。

## Installation

```sh
$ npm install cookie-parser
```

## API

```js
var express      = require('express')
var cookieParser = require('cookie-parser')

var app = express()
app.use(cookieParser())
```

### cookieParser(secret, options)

- `secret` 一个字符串用于签名cookies。这是可选的,如果没有指定,不会解析签名过的cookies。
- `options` 作为第二选项传递给 `cookie.parse`的对象. 查看 [cookie](https://www.npmjs.org/package/cookie) 获得更多信息.
  - `decode` 一个解码cookie的值的函数

### cookieParser.JSONCookie(str)

解析一个cookie值作为JSON cookie。如果是一个JSON cookie将返回解析JSON值,否则将返回传递的值。

### cookieParser.JSONCookies(cookies)

给定一个对象,这将遍历键和调用JSONCookie每个值。这将返回相同的对象。

### cookieParser.signedCookie(str, secret)

解析一个cookie值作为签名的cookie。如果是签名的cookie和签名是有效将返回解析后的无符号值,否则它将返回传递的值。

### cookieParser.signedCookies(cookies, secret)

给定一个对象,如果它是一个签名的cookie和签名是有效将遍历键和检查任何值是否签名过的cookie。的,关键是删除从对象并将其添加到返回的新对象。

## Example

```js
var express      = require('express')
var cookieParser = require('cookie-parser')

var app = express()
app.use(cookieParser())

app.get('/', function(req, res) {
  console.log("Cookies: ", req.cookies)
})

app.listen(8080)

// curl command that sends an HTTP request with two cookies
// curl http://127.0.0.1:8080 --cookie "Cho=Kim;Greet=Hello"
```

### [MIT Licensed](LICENSE)

[npm-image]: https://img.shields.io/npm/v/cookie-parser.svg
[npm-url]: https://npmjs.org/package/cookie-parser
[travis-image]: https://img.shields.io/travis/expressjs/cookie-parser/master.svg
[travis-url]: https://travis-ci.org/expressjs/cookie-parser
[coveralls-image]: https://img.shields.io/coveralls/expressjs/cookie-parser/master.svg
[coveralls-url]: https://coveralls.io/r/expressjs/cookie-parser?branch=master
[downloads-image]: https://img.shields.io/npm/dm/cookie-parser.svg
[downloads-url]: https://npmjs.org/package/cookie-parser
