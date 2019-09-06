# koa-static

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Dependency Status][david-image]][david-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]

 Koa static file serving middleware, wrapper for [`koa-send`](https://github.com/koajs/send).


## 源码解读

通过defer参数来决定next的执行位置，从而决定该中间件的执行时机。

然后底层基于koa-send模块的完成真正的静态资源文件服务逻辑。

核心源码如下：

``` js

'use strict'

/**
 * Module dependencies.
 */

const debug = require('debug')('koa-static')
const { resolve } = require('path')
const assert = require('assert')
const send = require('koa-send')

/**
 * Expose `serve()`.
 */

module.exports = serve

/**
 * Serve static files from `root`.
 *
 * @param {String} root
 * @param {Object} [opts]
 * @return {Function}
 * @api public
 */

function serve (root, opts) {
  opts = Object.assign({}, opts)

  assert(root, 'root directory is required to serve files')

  // options
  debug('static "%s" %j', root, opts)
  opts.root = resolve(root)
  if (opts.index !== false) opts.index = opts.index || 'index.html'
  // 通过defer参数来决定中间件的执行，next的位置。
  if (!opts.defer) {
    return async function serve (ctx, next) {
      let done = false
      // 如果是HEAD和GET请求，直接调用koa-send模块
      if (ctx.method === 'HEAD' || ctx.method === 'GET') {
        try {
          done = await send(ctx, ctx.path, opts)
        } catch (err) {
          if (err.status !== 404) {
            throw err
          }
        }
      }
      //完成后，走向下一中间件
      if (!done) {
        await next()
      }
    }
  }

  return async function serve (ctx, next) {
    await next()

    if (ctx.method !== 'HEAD' && ctx.method !== 'GET') return
    // response is already handled
    if (ctx.body != null || ctx.status !== 404) return // eslint-disable-line

    try {
      await send(ctx, ctx.path, opts)
    } catch (err) {
      if (err.status !== 404) {
        throw err
      }
    }
  }
}

```


## Installation

```bash
$ npm install koa-static
```

## API

```js
const Koa = require('koa');
const app = new Koa();
app.use(require('koa-static')(root, opts));
```

* `root` root directory string. nothing above this root directory can be served
* `opts` options object.

### Options

 - `maxage` Browser cache max-age in milliseconds. defaults to 0
 - `hidden` Allow transfer of hidden files. defaults to false
 - `index` Default file name, defaults to 'index.html'
 - `defer` If true, serves after `return next()`, allowing any downstream middleware to respond first.
 - `gzip`  Try to serve the gzipped version of a file automatically when gzip is supported by a client and if the requested file with .gz extension exists. defaults to true.
 - `br`  Try to serve the brotli version of a file automatically when brotli is supported by a client and if the requested file with .br extension exists (note, that brotli is only accepted over https). defaults to true.
 - [setHeaders](https://github.com/koajs/send#setheaders) Function to set custom headers on response.
 - `extensions` Try to match extensions from passed array to search for file when no extension is sufficed in URL. First found is served. (defaults to `false`)

## Example

```js
const serve = require('koa-static');
const Koa = require('koa');
const app = new Koa();

// $ GET /package.json
app.use(serve('.'));

// $ GET /hello.txt
app.use(serve('test/fixtures'));

// or use absolute paths
app.use(serve(__dirname + '/test/fixtures'));

app.listen(3000);

console.log('listening on port 3000');
```

### See also

 - [koajs/conditional-get](https://github.com/koajs/conditional-get) Conditional GET support for koa
 - [koajs/compress](https://github.com/koajs/compress) Compress middleware for koa
 - [koajs/mount](https://github.com/koajs/mount) Mount `koa-static` to a specific path

## License

  MIT

[npm-image]: https://img.shields.io/npm/v/koa-static.svg?style=flat-square
[npm-url]: https://npmjs.org/package/koa-static
[github-tag]: http://img.shields.io/github/tag/koajs/static.svg?style=flat-square
[github-url]: https://github.com/koajs/static/tags
[travis-image]: https://img.shields.io/travis/koajs/static.svg?style=flat-square
[travis-url]: https://travis-ci.org/koajs/static
[coveralls-image]: https://img.shields.io/coveralls/koajs/static.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/koajs/static?branch=master
[david-image]: http://img.shields.io/david/koajs/static.svg?style=flat-square
[david-url]: https://david-dm.org/koajs/static
[license-image]: http://img.shields.io/npm/l/koa-static.svg?style=flat-square
[license-url]: LICENSE
[downloads-image]: http://img.shields.io/npm/dm/koa-static.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/koa-static
[gittip-image]: https://img.shields.io/gittip/jonathanong.svg?style=flat-square
[gittip-url]: https://www.gittip.com/jonathanong/
