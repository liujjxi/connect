# [morgan](https://github.com/expressjs/morgan)
node.js中间件 HTTP请求日志记录器
> 德克斯特的名字命名,直到完成你才能看到。

## API

```js
var morgan = require('morgan')
```

### morgan(format, options)

使用给定的格式和选项创建一个新的morgan记录器中间件功能。forma参数可以是一系列预定义的名称(见下面的名字),格式字符串的字符串,或一个函数,它将产生一个日志条目。
  
#### Options

morgan接受这些属性的选择对象
  
#### immediate

写日志请求的响应。这意味着请求
被记录,即使服务器崩溃,_但数据响应(如
响应代码、内容长度等)不能登录_。

#### skip

确定日志被跳过的函数,默认值为 `false`。这个函数将当做`skip (req,res)`来调用。

```js
// EXAMPLE: only log error responses
morgan('combined', {
  skip: function (req, res) { return res.statusCode < 400 }
})
```

#### stream

写日志行输出流，默认为 `process.stdout`。

#### Predefined Formats

有各种预定义的格式提供:

##### combined

标准Apache日志输出相结合。

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"
```

##### common

标准Apache日志输出。

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length]
```


##### dev

简洁的响应状态输出彩色的开发使用。`:status`服务器错误代码使用红色，客户端错误使用黄色代码，重定向代码使用青色,所有其他代码用本色。

```
:method :url :status :response-time ms - :res[content-length]
```

##### short

短于默认的,也包括响应时间。

```
:remote-addr :remote-user :method :url HTTP/:http-version :status :res[content-length] - :response-time ms
```

##### tiny

最简洁的输出。

```
:method :url :status :res[content-length] - :response-time ms
```

#### Tokens

##### Creating new tokens

要定义一个tokens,只需调用`morgan.token()`的名称和一个回调函数。这个回调函数将返回一个字符串值。然后在这种情况可以返回值类型":type"下:

```js
morgan.token('type', function(req, res){ return req.headers['content-type']; })
```
调用morgan.token()使用相同的名称作为一个现有的令牌将覆盖,令牌的定义。

##### :date[format]

当前UTC日期和时间。可用的格式是:

  - `clf` for the common log format (`"10/Oct/2000:13:55:36 +0000"`)
  - `iso` for the common ISO 8601 date time format (`2000-10-10T13:55:36.000Z`)
  - `web` for the common RFC 1123 date time format (`Tue, 10 Oct 2000 13:55:36 GMT`)

如果没有给定的格式,那么默认的就是 `web`.

##### :http-version

HTTP请求的版本。

##### :method

HTTP请求的版本。

##### :referrer

介绍header的请求。如果存在拼写错误将使用标准的引用页头,否则上线。

##### :remote-addr

The remote address of the request. This will use `req.ip`, otherwise the standard `req.connection.remoteAddress` value (socket address).

##### :remote-user

The user authenticated as part of Basic auth for the request.

##### :req[header]

The given `header` of the request.

##### :res[header]

The given `header` of the response.

##### :response-time

The time between the request coming into `morgan` and when the response headers are written, in milliseconds.

##### :status

The status code of the response.

##### :url

The URL of the request. This will use `req.originalUrl` if exists, otherwise `req.url`.

##### :user-agent

The contents of the User-Agent header of the request.

## Examples

### express/connect

简单的应用程序,将所有请求登录Apache组合格式发送到STDOUT

```js
var express = require('express')
var morgan = require('morgan')

var app = express()

app.use(morgan('combined'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### vanilla http server

简单的应用程序,将所有请求登录Apache组合格式发送到STDOUT

```js
var finalhandler = require('finalhandler')
var http = require('http')
var morgan = require('morgan')

// create "middleware"
var logger = morgan('combined')

http.createServer(function (req, res) {
  var done = finalhandler(req, res)
  logger(req, res, function (err) {
    if (err) return done(err)

    // respond to request
    res.setHeader('content-type', 'text/plain')
    res.end('hello, world!')
  })
})
```

### write logs to a file

#### single file

简单的应用程序,将所有请求登录Apache组合格式文件`access.log`.

```js
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')

var app = express()

// create a write stream (in append mode)
var accessLogStream = fs.createWriteStream(__dirname + '/access.log', {flags: 'a'})

// setup the logger
app.use(morgan('combined', {stream: accessLogStream}))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

#### log file rotation

简单的应用程序,将所有请求登录Apache格式结合一个日志文件日期的在 `log/` 目录中使用 [file-stream-rotator module](https://www.npmjs.com/package/file-stream-rotator).

```js
var FileStreamRotator = require('file-stream-rotator')
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')

var app = express()
var logDirectory = __dirname + '/log'

// ensure log directory exists
fs.existsSync(logDirectory) || fs.mkdirSync(logDirectory)

// create a rotating write stream
var accessLogStream = FileStreamRotator.getStream({
  filename: logDirectory + '/access-%DATE%.log',
  frequency: 'daily',
  verbose: false
})

// setup the logger
app.use(morgan('combined', {stream: accessLogStream}))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### use custom token formats

示例应用程序将使用自定义令牌格式。这增加了ID的所有请求并使用`:id` 显示。

```js
var express = require('express')
var morgan = require('morgan')
var uuid = require('node-uuid')

morgan.token('id', function getId(req) {
  return req.id
})

var app = express()

app.use(assignId)
app.use(morgan(':id :method :url :response-time'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})

function assignId(req, res, next) {
  req.id = uuid.v4()
  next()
}
```

## License

[MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/morgan.svg
[npm-url]: https://npmjs.org/package/morgan
[travis-image]: https://img.shields.io/travis/expressjs/morgan/master.svg
[travis-url]: https://travis-ci.org/expressjs/morgan
[coveralls-image]: https://img.shields.io/coveralls/expressjs/morgan/master.svg
[coveralls-url]: https://coveralls.io/r/expressjs/morgan?branch=master
[downloads-image]: https://img.shields.io/npm/dm/morgan.svg
[downloads-url]: https://npmjs.org/package/morgan
[gratipay-image]: https://img.shields.io/gratipay/dougwilson.svg
[gratipay-url]: https://www.gratipay.com/dougwilson/
