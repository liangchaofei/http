# 02-HTTP各种特性

**跨域**


看下面代码
1.server.js
```javascript
// server.js
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  const html = fs.readFileSync('test.html', 'utf8')
  response.writeHead(200, {
    'Content-Type': 'text/html'
  })
  response.end(html)
}).listen(8888)

console.log('server listening on 8888')
```


2.server2.js
```javascript
// server2.js
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```


3.test.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<!-- cors1 -->
<script>
  var xhr = new XMLHttpRequest()
  xhr.open('GET', 'http://127.0.0.1:8887/')
  xhr.send()
</script>
</html>
```
从上面代码中可以看出，server.js中去读取test.html中内容，跑在8888端口
在test.html中，向8887端口发送http请求。


打开控制台看到：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367645113-e4023c0e-99ba-42e5-9cf0-63ae7cc51b47.png#align=left&display=inline&height=143&margin=%5Bobject%20Object%5D&name=image.png&originHeight=286&originWidth=1104&size=63962&status=done&style=none&width=552)
这就是产生了跨域。


通过cors解决跨域，在server2.js中添加下面内容 Access-Control-Allow-Origin 头
```javascript
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  response.writeHead(200, {
    'Access-Control-Allow-Origin': '*',
  })
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```
'Access-Control-Allow-Origin': '*', 代表允许所有第三方请求。但是实际中不会，不安全。
重新打开控制台，不会出现报错了。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367676293-f5ef3a0f-fa62-4cee-af9b-fa70fcc06a46.png#align=left&display=inline&height=89&margin=%5Bobject%20Object%5D&name=image.png&originHeight=178&originWidth=1104&size=18699&status=done&style=none&width=552)
我们也可以通过jsonp来解决跨域问题。
修改下test.html  将请求放到script标签中的src属性中。这是因为浏览器允许script link这些标签跨域。
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<script src='http://127.0.0.1:8887/'>
</script>
</html>
```






**CORS预请求**


CORS预请求：
1.跨域的时候，默认允许的方法只有get post head请求。 其他请求会有预请求。
2.Content-Type中除了text/plain ，multipart/form-data， application/x-www-form-urlencoded ，其他都会有预请求。
3.header头限制




我们自定义下header，修改下test.html
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<!-- cors1 -->
<!-- <script>
  fetch('http://localhost:8887', {
    method: 'POST',
    headers: {
      'X-Test-Cors': '123'
    }
  })
</script> -->
</html>
```
打开控制台如下报错：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367720837-6904b0cb-06d6-4bef-8ac1-22aadb29aa34.png#align=left&display=inline&height=165&margin=%5Bobject%20Object%5D&name=image.png&originHeight=330&originWidth=1112&size=73805&status=done&style=none&width=556)
意思是自定义header，不允许跨域。这也就是CORS预请求。


可以看请求，代码中发送的是POST请求，但是图中发送的是options请求
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367762616-8e0bde3e-08d7-4155-ba5d-bc2e8c3c5ee4.png#align=left&display=inline&height=151&margin=%5Bobject%20Object%5D&name=image.png&originHeight=302&originWidth=1024&size=52842&status=done&style=none&width=512)
可以通过在服务端中添加对应的header头：
```javascript
// server2.js
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  response.writeHead(200, {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'X-Test-Cors', // 添加自定义头
  })
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```
重新运行，看下浏览器。此时发送了两个请求，先是options请求，再是post请求。控制台也不会报错了。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367887948-7817b394-39b7-4cf7-bfb1-f0ab32594f13.png#align=left&display=inline&height=437&margin=%5Bobject%20Object%5D&name=image.png&originHeight=874&originWidth=1416&size=150858&status=done&style=none&width=708)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367902160-9a41de81-f9bd-40b2-97c2-2c23d2a4946e.png#align=left&display=inline&height=486&margin=%5Bobject%20Object%5D&name=image.png&originHeight=972&originWidth=1426&size=165194&status=done&style=none&width=713)


下面再试下请求方式的预请求。
我们修改下test.html中的请求方式，将POST改为PUT。重新允许，看下浏览器，控制台报错，只发送了options请求。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367947753-7a4e0792-519d-4bd3-a58e-2d14dc8c9f69.png#align=left&display=inline&height=184&margin=%5Bobject%20Object%5D&name=image.png&originHeight=368&originWidth=1508&size=92295&status=done&style=none&width=754)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605367964302-6cf686e5-bb9d-4475-acc0-915e6d45700b.png#align=left&display=inline&height=327&margin=%5Bobject%20Object%5D&name=image.png&originHeight=654&originWidth=1510&size=116387&status=done&style=none&width=755)


我们可以通过在服务端添加允许的请求方式头
```javascript
// server2.js
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  response.writeHead(200, {
    'Access-Control-Allow-Origin': 'http://127.0.0.1:8888',
    'Access-Control-Allow-Headers': 'X-Test-Cors',
    'Access-Control-Allow-Methods': 'POST, PUT, DELETE',
  })
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```
重新运行。此时再看下浏览器。控制台不会报错。先有options请求，再有put请求。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605368006211-c31591d9-f2b5-4764-88a1-46f51c96742e.png#align=left&display=inline&height=395&margin=%5Bobject%20Object%5D&name=image.png&originHeight=790&originWidth=1512&size=139241&status=done&style=none&width=756)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605368019289-e1e94868-6c88-49aa-ad5d-3646a43d56f7.png#align=left&display=inline&height=425&margin=%5Bobject%20Object%5D&name=image.png&originHeight=850&originWidth=1430&size=144437&status=done&style=none&width=715)


下面我们再添加一个max-age头，如下：
```javascript
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  response.writeHead(200, {
    'Access-Control-Allow-Origin': 'http://127.0.0.1:8888',
    'Access-Control-Allow-Headers': 'X-Test-Cors',
    'Access-Control-Allow-Methods': 'POST, PUT, DELETE',
    'Access-Control-Max-Age': '1000'
  })
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```
这个头代表，开始会先触发options请求，再触发put请求。1s过后，只有put请求。也就是告诉浏览器这个请求不用再预请求了。








**缓存**
**
可缓存性：
Cache-Control
1.public：整个http请求节点都可以缓存，包括代理缓存
2.private：只有发起的浏览器才可以缓存
3.no-cache：可以在本地缓存，但是需要服务器认证
4.no-store：任何节点都不可缓存


到期：
max-age = <seconds> 表示缓存多少s过期
s-max-age = <seconds> 可以代替max-age 但是只在代理服务器上才生效。在代理服务器，两个同时存在，会读取s-max-age




下面看下max-age示例:
test.html如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<script src="/script.js"></script>
</html>
```
server.js如下
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/script.js') {
    response.end('console.log("script loaded")')
  }
}).listen(8888)

console.log('server listening on 8888')
```
允许node server.js。打开控制台，可以看到：打印了 script loaded
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605407842403-cdb2f6ea-5dfd-4fd4-917a-3992ef7d70c0.png#align=left&display=inline&height=187&margin=%5Bobject%20Object%5D&name=image.png&originHeight=374&originWidth=1568&size=36685&status=done&style=none&width=784)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605407865358-8a5dcd39-596c-401c-8a78-978545318f77.png#align=left&display=inline&height=208&margin=%5Bobject%20Object%5D&name=image.png&originHeight=416&originWidth=1230&size=71550&status=done&style=none&width=615)


![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605407888862-b7765271-d228-44ac-a2e2-f8f4cecf37e2.png#align=left&display=inline&height=312&margin=%5Bobject%20Object%5D&name=image.png&originHeight=624&originWidth=1726&size=108026&status=done&style=none&width=863)


我们在server.js中加入Cache-Control : max-age =  20 来缓存
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/script.js') {
    response.writeHead(200, {
      'Content-Type': 'text/javascript',
      'Cache-Control': 'max-age=20'
    })
    response.end('console.log("script loaded")')
  }
}).listen(8888)

console.log('server listening on 8888')
```
第一次刷新：看到size为228B，时间是3ms
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605407964980-8151b17a-028b-4162-b72e-41c14f3a56fc.png#align=left&display=inline&height=319&margin=%5Bobject%20Object%5D&name=image.png&originHeight=638&originWidth=1906&size=118806&status=done&style=none&width=953)


过了20s后，再次刷新，可以看到这次是从memory cache中读取，而且时间是0ms
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605408077961-be685da1-49f2-4aa2-863b-943634c25e86.png#align=left&display=inline&height=343&margin=%5Bobject%20Object%5D&name=image.png&originHeight=686&originWidth=1946&size=116801&status=done&style=none&width=973)


接下来，我看下打印的内容：
1.在不加入cacae-controle情况下，打印如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605408147717-91a5bbd4-13e4-4ed6-8652-0fdba6e6fe73.png#align=left&display=inline&height=105&margin=%5Bobject%20Object%5D&name=image.png&originHeight=210&originWidth=1966&size=34770&status=done&style=none&width=983)


2.加入cache-controle，20s内，打印如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605408214361-91271a44-1229-4dac-b118-35ee2e97ea9b.png#align=left&display=inline&height=121&margin=%5Bobject%20Object%5D&name=image.png&originHeight=242&originWidth=1928&size=35796&status=done&style=none&width=964)
3.修改打印的内容，因为设置的max-age为20s，缓存时间过去了，所以打印新内容，打印如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605408624046-66a0242d-3e4f-4259-b3ce-a6d79264996e.png#align=left&display=inline&height=103&margin=%5Bobject%20Object%5D&name=image.png&originHeight=206&originWidth=522&size=8713&status=done&style=none&width=261)
4.我们将max-age改为200，将内容改回原先内容，重新运行打印如下：说明还是从缓存读取
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605408780920-ddb5854a-b009-4c56-b162-5fadfb85fd92.png#align=left&display=inline&height=123&margin=%5Bobject%20Object%5D&name=image.png&originHeight=246&originWidth=652&size=11335&status=done&style=none&width=326)

实际工作中，cache-control是客户端缓存，一般将max-age设置很长时间。但是项目内容改变了，因为缓存没有更新。此时一般会使用时间戳或者内容hash来更新项目内容。




下面看下缓存的流程：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605409330007-6b6dab63-b34b-4913-b969-bc8687dbaba2.png#align=left&display=inline&height=543&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1086&originWidth=2282&size=420855&status=done&style=none&width=1141)


验证能否使用缓存的头：
1.Last-Modified
2.ETag


Last-Modified：
1.上次修改时间
2.配合If-Modified-Since使用
3.对比上次修改时间来验证资源是否需要更新    




ETag：
1.数据签名
2.内容改变，数据签名改变，对比两个数据签名是否一样，从而判断是否缓存
3.配合If-Match使用

下面通过代码演示一下：
test.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<script src="/script.js"></script>
</html>
```
server.js
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/script.js') {
      response.writeHead(200, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
      })
      response.end('console.log("script loaded twice")')
  }
}).listen(8888)

console.log('server listening on 8888')
```
代码中设置了Cache-Control: max-age=2000000, no-cache ,  'Last-Modified': '123', 'Etag': '777'
第一次刷新页面看下,在response header中返回：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605423668255-ef9e8e01-b7da-4b36-96da-731df21d4cc1.png#align=left&display=inline&height=490&margin=%5Bobject%20Object%5D&name=image.png&originHeight=980&originWidth=1874&size=179880&status=done&style=none&width=937)


当第二次刷新的时候，request header就会分别带上对应的header，来判断是否命中缓存
可以看到，Etag和**If-None-Match一样，Last-Modified 和If-Modified-Since 一样。**
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605423790782-a529d9a3-95c3-4424-8726-d863421eb64d.png#align=left&display=inline&height=546&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1092&originWidth=2008&size=247920&status=done&style=none&width=1004)


但是，第二次刷新还是请求了服务器，返回200
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605423905720-b05aa6a7-f978-4adb-b44a-2a72344a2f28.png#align=left&display=inline&height=148&margin=%5Bobject%20Object%5D&name=image.png&originHeight=296&originWidth=1840&size=48093&status=done&style=none&width=920)


此时，可以通过在服务端判断Etag来确定是否可以读取缓存
修改server.js如下
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/script.js') {
    
    const etag = request.headers['if-none-match']
    if (etag === '777') {
      response.writeHead(304, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
      })
      response.end()
    } else {
      response.writeHead(200, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
      })
      response.end('console.log("script loaded twice")')
    }
  }
}).listen(8888)

console.log('server listening on 8888')
```


此时重新运行，如下：
代码中当命中Etag策略的时候，服务端没有返回内容，浏览器没有去请求服务器，而是返回304，内容确返回了缓存中的内容。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424072408-f5bc0efc-b1af-4e94-b799-73b9d60b8073.png#align=left&display=inline&height=524&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1048&originWidth=1984&size=196464&status=done&style=none&width=992)


现在把server.js中的no-cache去掉，重新运行。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424284699-2254ccba-f50d-4a44-b9ab-89244292c39c.png#align=left&display=inline&height=516&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1032&originWidth=1556&size=167322&status=done&style=none&width=778)
控制台打印如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424357765-10791ebe-48e3-470f-a857-6a17fad24f1e.png#align=left&display=inline&height=120&margin=%5Bobject%20Object%5D&name=image.png&originHeight=240&originWidth=1920&size=22609&status=done&style=none&width=960)


当再次刷新：是从缓存中读取内容。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424392210-58680c9d-6424-4bb3-89e7-bd6a7a68ca18.png#align=left&display=inline&height=328&margin=%5Bobject%20Object%5D&name=image.png&originHeight=656&originWidth=1904&size=120301&status=done&style=none&width=952)


现在将no-cache改为no-store，打印如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424484572-8ffb498d-ceb2-4f0f-bf01-59c8f56e1ef9.png#align=left&display=inline&height=493&margin=%5Bobject%20Object%5D&name=image.png&originHeight=986&originWidth=1514&size=152174&status=done&style=none&width=757)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424467548-f5e5bfef-effa-450c-bc22-e9be8972461f.png#align=left&display=inline&height=180&margin=%5Bobject%20Object%5D&name=image.png&originHeight=360&originWidth=2014&size=68463&status=done&style=none&width=1007)


当再次刷新：并没有从缓存中读取。这也就是no-cache和no-store的区别。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605424507919-cb925fc0-ac11-4b83-9dc6-9703f6e9b87e.png#align=left&display=inline&height=319&margin=%5Bobject%20Object%5D&name=image.png&originHeight=638&originWidth=1862&size=118383&status=done&style=none&width=931)






**cookie**
1.通过Set-Cookie设置
2.下次请求就会带上
3.max-age和expires设置过期时间
4.Secure只在https的时候发送
5.HttpOnly 无法通过 document.cookie访问

下面代码演示下：
test.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div>Content</div>
</body>
<script>
  console.log(document.cookie)
</script>
</html>
```
server.js
```html
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html',
      'Set-Cookie': 'id=123'
    })
    response.end(html)
  }

}).listen(8888)

console.log('server listening on 8888')
```
在服务端通过Set-Cookie设置了id=123，当首次访问的时候，可以看到response header中显示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605430906386-ae78e486-a6a9-4d61-ba94-db77543be65a.png#align=left&display=inline&height=513&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1026&originWidth=1542&size=181216&status=done&style=none&width=771)


同时控制台，也能打印出cookie
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605430932620-2138f267-0c47-42f4-b4e2-10f042b93cf0.png#align=left&display=inline&height=71&margin=%5Bobject%20Object%5D&name=image.png&originHeight=142&originWidth=1516&size=15719&status=done&style=none&width=758)


当第二次访问的时候，request header 就会自动带上cookie，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431024510-ef357238-f1b8-4e36-80a4-a9074821380f.png#align=left&display=inline&height=482&margin=%5Bobject%20Object%5D&name=image.png&originHeight=964&originWidth=1566&size=170367&status=done&style=none&width=783)




也可以设置多个cookie，修改server.js中代码如下:
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html',
      'Set-Cookie': ['id=123','abc=456']
    })
    response.end(html)
  }

}).listen(8888)

console.log('server listening on 8888')
```
重新运行，可以看到：设置了两个cookie
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431110174-19189a12-5030-4676-8381-3266f6d6180f.png#align=left&display=inline&height=457&margin=%5Bobject%20Object%5D&name=image.png&originHeight=914&originWidth=1530&size=152887&status=done&style=none&width=765)


同样第二次访问的时候，request header自动带上所有cookie，多个cookie以分号隔开。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431155102-37b1f562-02d9-433b-b90f-3fbb693339c0.png#align=left&display=inline&height=508&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1016&originWidth=1568&size=186735&status=done&style=none&width=784)


通过max-age给cookie设置过期时间，修改代码如：给id=123，设置过期时间为2s
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html',
      'Set-Cookie': ['id=123; max-age=2', 'abc=456']
    })
    response.end(html)
  }

}).listen(8888)

console.log('server listening on 8888')
```
重新运行，第一次打开如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431286116-603f3dbd-bf96-46d1-a209-d225bd13950c.png#align=left&display=inline&height=465&margin=%5Bobject%20Object%5D&name=image.png&originHeight=930&originWidth=1450&size=162961&status=done&style=none&width=725)


在2s内，重新刷新浏览器打开：此时有id=123


![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431372798-9a2374c6-8bf7-4cc4-a7ee-7d274b015587.png#align=left&display=inline&height=459&margin=%5Bobject%20Object%5D&name=image.png&originHeight=918&originWidth=1542&size=164616&status=done&style=none&width=771)


在2s后，重新刷新浏览器打开：此时只有abc=456，没有id=123了
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605431416862-365ba93b-ead4-4eeb-a948-6ad0eaa552d4.png#align=left&display=inline&height=470&margin=%5Bobject%20Object%5D&name=image.png&originHeight=940&originWidth=1546&size=167752&status=done&style=none&width=773)


我们知道，不同域名，cookie不能共享。但是可以做到二级域名共享cookie。使用domain。
在abc=456 这个cookie设置domain，表示二级域名是test.com，可以共享cookie
```javascript
'Set-Cookie': ['id=123; max-age=2', 'abc=456;domain=test.com']
```




**长链接：**
浏览器发送一个http请求，http1.0会直接关闭tcp链接。现在发送http请求，和服务端协商是否关闭TCP链接，
使用connection：keep-alive 来保持tcp不关闭。现在浏览器默认是keep-alive


代码演示：
test.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <img src="/test1.jpg" alt="">
  <img src="/test2.jpg" alt="">
  <img src="/test3.jpg" alt="">
  <img src="/test4.jpg" alt="">
  <img src="/test5.jpg" alt="">
  <img src="/test6.jpg" alt="">
  <img src="/test7.jpg" alt="">
  <img src="/test11.jpg" alt="">
  <img src="/test12.jpg" alt="">
  <img src="/test13.jpg" alt="">
  <img src="/test14.jpg" alt="">
  <img src="/test15.jpg" alt="">
  <img src="/test16.jpg" alt="">
  <img src="/test17.jpg" alt="">
  <img src="/test111.jpg" alt="">
  <img src="/test112.jpg" alt="">
  <img src="/test113.jpg" alt="">
  <img src="/test114.jpg" alt="">
  <img src="/test115.jpg" alt="">
  <img src="/test116.jpg" alt="">
  <img src="/test117.jpg" alt="">
</body>
</html>
```
server.js
```javascript
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  const html = fs.readFileSync('test.html', 'utf8')
  const img = fs.readFileSync('test.jpg')
  if (request.url === '/') {
    response.writeHead(200, {
      'Content-Type': 'text/html',
    })
    response.end(html)
  } else {
    response.writeHead(200, {
      'Content-Type': 'image/jpg',
      'Connection': 'keep-alive' // or close
    })
    response.end(img)
  }

}).listen(8888)

console.log('server listening on 8888')
```
首先把浏览器设置如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605432993824-80cf3443-bac2-4d99-856f-6fe9be725f85.png#align=left&display=inline&height=486&margin=%5Bobject%20Object%5D&name=image.png&originHeight=972&originWidth=2108&size=157992&status=done&style=none&width=1054)


刷新浏览器：可以看到前两个connection ID一样，与网络环境也有关系。


![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605433037617-10e47a97-c86c-46b5-b084-4216917c9f7b.png#align=left&display=inline&height=612&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1224&originWidth=2060&size=472910&status=done&style=none&width=1030)
将connection设置为close，即可关闭长链接。


**重定向**
301：永久重定向
302：临时重定向


通过location头设置重定向


代码演示：

```javascript
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    response.writeHead(302, {  // or 301
      'Location': '/new'
    })
    response.end()
  }
  if (request.url === '/new') {
    response.writeHead(200, {
      'Content-Type': 'text/html',
    })
    response.end('<div>this is content</div>')
  }
}).listen(8888)

console.log('server listening on 8888')
```
打开浏览器可以看到: 先访问根路径，通过location，重定向到/new路径
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605433870289-f71ceeaa-0008-4559-a621-4e2b69426d99.png#align=left&display=inline&height=628&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1256&originWidth=2382&size=260386&status=done&style=none&width=1191)


再看下301和302，


设置为302，第一次访问浏览器，看到先访问根路径，再访问/new
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605434119400-17d7356f-3221-46d3-ba0b-697d2f5cb8ff.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1890&size=111189&status=done&style=none&width=945)


刷新浏览器，同样是先访问根路径，再访问/new
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605434149875-ea75683d-e0e7-47bf-b6fc-48f340e6ded5.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1890&size=111189&status=done&style=none&width=945)

设置为301，首次访问，先访问根路径，再访问/new
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605434175364-1d1b6a7c-fa73-4e42-8725-267fdc94ad76.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1890&size=111189&status=done&style=none&width=945)


再次刷新，访问，直接访问/new路径
![image.png](https://cdn.nlark.com/yuque/0/2020/png/421731/1605434200877-c98474ae-1f00-4ddb-bff1-8048cbcc8e36.png#align=left&display=inline&height=68&margin=%5Bobject%20Object%5D&name=image.png&originHeight=136&originWidth=1186&size=19498&status=done&style=none&width=593)




**数据协商**
Request header:
Accept：请求什么类型内容
Accept-Encoding：压缩方式
Accept-Language：语言
User-Agent：用户浏览器信息


Response Header:
Content：返回的内容
Content-Type：类型
Content-Encoding：压缩方式
Content-Language：语言

**CSP**


[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy/](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)
**
