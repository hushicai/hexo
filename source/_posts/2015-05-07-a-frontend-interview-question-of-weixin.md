title: "微信的一道前端面试题"
date: 2015-05-07 23:03:28
tags:
s: a frontend interview question of weixin
---

今天听说了一道微信的前端面试题，内容大概如下：

> 实现一个LazyMan，可以按照以下方式调用:
> LazyMan("Hank")输出:
> Hi! This is Hank!
> 
> LazyMan("Hank").sleep(10).eat("dinner")输出
> Hi! This is Hank!
> //等待10秒..
> Wake up after 10
> Eat dinner~
> 
> LazyMan("Hank").eat("dinner").eat("supper")输出
> Hi This is Hank!
> Eat dinner~
> Eat supper~
> 
> LazyMan("Hank").sleepFirst(5).eat("supper")输出
> //等待5秒
> Wake up after 5
> Hi This is Hank!
> Eat supper
> 
> 以此类推。

咦，似曾相识耶，貌似是经典的异步编程问题。

<!-- more -->

一个简单的做法就是使用队列+next。

```javascript
function LazyMan(name) {
    var queue = [];
    var task = {
        wait: function (second) {
            return function () {
                setTimeout(function () {
                    console.log('Wake up after ' + second);
                    next();
                }, second * 1000);
            };
        },
        eat: function (part) {
            return function () {
                console.log('Eat ' + part + '~');
                next();
            };
        },
        hi: function () {
            console.log('Hi! This is ' + name + '!');
            next();
        }
    };

    queue.push(task.hi);

    function next() {
        var fn = queue.shift();
        fn && fn();
    }

    // flush later
    setTimeout(function () {
        next();
    }, 0);

    return {
        sleep: function (second) {
            queue.push(task.wait(second));
            return this;
        },
        sleepFirst: function (second) {
            queue.unshift(task.wait(second));
            return this;
        },
        eat: function (part) {
            queue.push(task.eat(part));
            return this;
        }
    };
}

exports.LazyMan = LazyMan;
```

测试：

```javascript
var LazyMan = require('./LazyMan');
LazyMan('Hank').sleepFirst(10).eat('breadfast').sleep(5).eat('lunch').sleep(10).eat('dinner');
```

该问题也可以使用Promise来解决。

```javascript
function createWaitPromise(second) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('Wake up after ' + second);
        }, second * 1000);
    });
}

function LazyMan(name) {
    var p = new Promise(function (resolve, reject) {
        resolve('Hi! This is ' + name + '!');
    });

    return {
        sleep: function (second) {
            p = p.then(function (msg) {
                console.log(msg);
                return createWaitPromise(second);
            });
            return this;
        },
        sleepFirst: function (second) {
            var op = p;
            p = createWaitPromise(second).then(function (msg) {
                console.log(msg);
                return op;
            });
            return this;
        },
        eat: function (part) {
            var pn = new Promise(function (resolve) {
                resolve('Eat ' + part + '~');
            });
            p = p.then(function (msg) {
                console.log(msg);
                return pn;
            });
            return this;
        },
        print: function () {
            return p.then(function (msg) {
                console.log(msg);
            });
        }
    };
}

exports.LazyMan = LazyMan;
```

测试：

```javascript
var LazyMan = require('./LazyMan');
LazyMan('Hank').sleepFirst(5).eat('breadfast').sleep(5).eat('lunch').sleep(5).eat('dinner').print();
```

print是用来取出最后一个promise的消息。

以上测试都在nodejs环境跑，欢迎拍砖~！
