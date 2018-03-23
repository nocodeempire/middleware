## 对于时间过长的日志输出
```js
var connect = require('connect'),
morgan = require('morgan');
time = require('time')
var app = connect();
app.use(morgan('dev'))
app.use(time({time: 500}));
app.listen(9001)

app.use('/a', (req, res, next) => {
    res.end("a")
})
app.use('/b', (req, res, next) => {
    setTimeout(function() {
        res.end("b")
    },1000) 
})
```
### 时间中间件 time
```js
module.exports = function(ops) {
    var time = ops.time || 100;
    return function(req, res, next) {
        var timer = setTimeout(function() {
            // 日志写入文件,但没有做文件的创建,需自己创建文件
            // require('fs').appendFile('log.txt', "花了太长时间"+req.originalUrl+"\r")
            console.log("花了太长时间",req.method,req.url)
        },time)
        //保持对原始函数的引用(var end = res.end), 然后在重写的函数中恢复原始函数在调用它
        var end = res.end;
        res.end = function(chunk, encoding) {
            res.end = end;
            clearTimeout(timer);
            res.end(chunk, encoding);
        }
        next();
    }
}
```
