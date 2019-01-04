代码注释

1.程序入口：
```
var server = http.createServer(function(req, res){
    routePath(req, res);
  });
```
当一个请求进来之后，进入到`http.createServer`这个逻辑里面，然后会调用` routePath(req, res)`这个函数去处理这个请求和响应。


2.function routePath(req, res){}：



>`url.parse(req.url, true) ：`解析url

>`routes[pathObj.pathname] ：`得到 pathname，也就是路由。

得到` pathname` 后，从 `routes `里根据` pathname`， 输入`pathname `里面 的 key 去匹配看能不能得到它的值，如果` pathname `这里面没有，那就会得到` undefined`，所以` handleFn `就是` undefined`。

如果` routes `里找不到的话，就进入 else 的逻辑让它去把这个请求当成静态文件去处理 `staticRoot`。
```
else{
staticRoot(path.resolve(__dirname, 'static'), req, res);
}
```

如果` routes `里找到了，匹配上了，就进入到这个逻辑：
```
req.query = pathObj.query;
var body = '';
req.on('data', function(chunk){
body += chunk;
}).on('end', function(){
req.body = parseBody(body);
handleFn(req, res);
});
```

我们把` url `请求的` query` 放到 `req `上` req.query = pathObj.query`，这样的话我们在` routes `中的地方就可以直接使用 `req.query`

```
{
    '/a': function(req, res){
      res.end(JSON.stringify(req.query));
    },

```
对于 POST 形式的东西，因为 POST 请求的数据它并不在 url 里面，所以它有一些特殊。我们可以通过` req.on('data', ...)` 就可以得到里面的数据；
然后 `req.on('end', ...)` 这个数据就全都完成了，就可以通过调用` parseBody(body)`把这个数据给解析出来，解析成一个对象然后放在 `body（ req.body）`上；
```
req.body = parseBody(body)

parseBody(body) 这个 body 得到的值其实就是 key = value，a = b，就是 key = value & key = value 的形式，我们把这个格式变成一个对象：
function parseBody(body){
    console.log(body);
    var obj = {};
    body.split('&').forEach(function(str){
      obj[str.solit('=')[0]] = str.split('=')[1];
    });
    return obj;
  }
```
得到` body `这个对象之后我们就可以调用 `handleFn(req, res)` 进入到对应的逻辑下。
换句话说，当用户调用的是` '/a' `，又有 GET 也有 POST 数据，那它就会执行` '/a' `的函数。
`req.query `就是 GET 对应的数据，`req.body` 就是 POST 对应的数据。那我们就可以直接获取里面的东西，比如说` req.body.username`。在这里得到数据之后，我们就可以模拟这个数据，就可以 mock 了。 就可以做各种判断了。

>nodejs 中间件：就是处理路由的。
>中间件：即在中间放一个物件，处理你的请求。


