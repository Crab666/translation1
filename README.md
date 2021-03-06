## dubbo2.js

![love dubbo](https://raw.githubusercontent.com/dubbo/dubbo2.js/master/resources/dubbo-love.png)

[![NPM](https://nodei.co/npm/dubbo2.js.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/dubbo2.js)

> dubbo2.js = Nodejs connected Java Dubbo service by dubbo protocol (dubbo head + hessian body)

多年期盼，一朝梦圆！ We love dubbo 👏

感谢 [js-to-java](https://github.com/node-modules/js-to-java),[hessian.js](https://github.com/node-modules/hessian.js) 两大核心模块, 感谢[fengmk2](https://github.com/fengmk2)和[dead-horse](https://github.com/dead-horse)老师。

## Features

1.  Keep it Simple (build tools for humans).

2.  Support zookeeper as register center

3.  TCP Dubbo Native protocol （Dubbo Header + Hessian Body）

4.  Socket Container (ServerAgent -> SocketWorker)

5.  Support Directly Dubbo (const Dubbo = DirectlyDubbo({..}))

6.  Middleware the same as Koa middleware, Easy to extend.

7.  Tracing (runtime info, call stack)

8.  Supported Dubbox

9.  Typescript type definition

10. Convert java dubbo interface to typescript module by interpret tools

11. SocketWorker was disconnected auto retry

## Getting Started

```shell
yarn add dubbo2.js
```

## How to Usage?

### dubbo2.js@2.0.4+

```typescript
//=====================service.ts==================
//generated by interpret tools
import {BasicTypeProvider} from './providers/com/alibaba/dubbo/demo/BasicTypeProvider';
import {DemoProvider} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {ErrorProvider} from './providers/com/alibaba/dubbo/demo/ErrorProvider';

export default {
  BasicTypeProvider,
  DemoProvider,
  ErrorProvider,
};

//===============dubbo.ts========================
import {Dubbo} from 'dubbo2.js';
import service from './service';

//创建dubbo对象
const dubbo = new Dubbo<typeof service>({
  application: {name: 'node-dubbo'},
  //zookeeper address
  register: 'localhost:2181',
  service,
});

//main method
(async () => {
  let {res, err} = await dubbo.service.DemoProvider.sayHello('node');
  //print {err: null, res:'hello node from dubbo service'}
  ({res, err} = await dubbo.service.DemoProvider.echo());
  //print {err: null, res: 'pang'}
  ({res, err} = await dubbo.service.DemoProvider.getUserInfo());
  //status: 'ok', info: { id: '1', name: 'test' }
})();
```

## 不是从 java 自动生成的 typescript 代码，如何注入到 dubbo 对象中？

```typescript
//创建要注入的service
import {Dubbo} from 'dubbo2.js';
const demoProvider = dubbo =>
  dubbo.proxyService({
    dubboInterface: 'com.alibaba.dubbo.demo.DemoProvider',
    version: '1.0.0',
    methods: {
      sayHello(name) {
        return [java.String(name)];
      },

      echo() {},

      test() {},

      getUserInfo() {
        return [
          java.combine('com.alibaba.dubbo.demo.UserRequest', {
            id: 1,
            name: 'nodejs',
            email: 'node@qianmi.com',
          }),
        ];
      },
    },
  });

//将该service合入dubbo对象构造函数的service对象中
const service = {
  demoProvider,
};

const dubbo = new Dubbo<typeof service>({
  //  ....other parameters
  service,
});
```

### dubbo2.js@1.xxx

```typescript
import {Dubbo, java, TDubboCallResult} from 'dubbo2.js';
//generated by interpret tools
import {BasicTypeProvider} from './providers/com/alibaba/dubbo/demo/BasicTypeProvider';
import {DemoProvider} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {ErrorProvider} from './providers/com/alibaba/dubbo/demo/ErrorProvider';

//创建dubbo对象
const dubbo = new Dubbo({
  application: {name: 'node-dubbo'},
  //zookeeper address
  register: 'localhost:2181',
  interfaces: [
    'com.alibaba.dubbo.demo.DemoProvider',
    'com.alibaba.dubbo.demo.BasicTypeProvider',
    'com.alibaba.dubbo.demo.ErrorProvider',
  ],
});

const basicTypeProvider = BasicTypeProvider(dubbo);
const demoProvider = DemoProvider(dubbo);
const errorProvider = ErrorProvider(dubbo);

//main method
(async () => {
  let {res, err} = await demoProvider.sayHello('node');
  //print {err: null, res:'hello node from dubbo service'}
  ({res, err} = await demoProvider.echo());
  //print {err: null, res: 'pang'}
  ({res, err} = await demoProvider.getUserInfo());
  //status: 'ok', info: { id: '1', name: 'test' }
})();
```


## as developer

```sh
brew install zookeeper
brew services start zookeeper

#运行java/dubbo-demo-provider下面的例子

yarn run test

# 全链路日志跟踪
DEBUG=dubbo* yarn run test
```

## dubbo flow graph

![dubbo-flow](https://raw.githubusercontent.com/dubbo/dubbo2.js/master/resources/dubbo2-flow.png)

## API

### create dubbo object

```javascript
const dubbo = new Dubbo({
  isSupportedDubbox     //是否支持dubbox - boolean, 默认false, 可选
  application           //记录应用的名称 - zookeeper的调用时候写入consumer, 类型({name: string};) 可选
  dubboInvokeTimeout    //设置dubbo调用超时时间 - number类型, 默认10s, 可选
  dubboSocketPool       //设置dubbo创建socket的pool大小 - 类型number, 默认4, 可选
  register              //设置zookeeper注册中心地址 - 类型string, 必填
  zkRoot                //zk的默认根路径 - 类型string, 默认/dubbo
  interfaces            //设置zk监听的接口名称 - 类型 Array<string>, 必填, 在dubbo2.js@2.0.4+版本中不在使用这个参数
  service               //注入到dubbo容器的dubbo服务 - 类型Object, 在dubbo2.js@2.0.4+使用
});

// Or
const dubbo = Dubbo.from({
  isSupportedDubbox     //是否支持dubbox - 类型boolean, 默认false, 可选
  application           //记录应用的名称 - zookeeper的调用时候写入consumer, 类型({name: string};) 可选
  dubboInvokeTimeout    //设置dubbo调用超时时间 - number类型, 默认10s, 可选
  dubboSocketPool       //设置dubbo创建socket的pool大小 - 类型number, 默认4, 可选
  register              //设置zookeeper注册中心地址 - 类型string, 必填
  zkRoot                //zk的默认根路径 - 类型string, 默认/dubbo
  interfaces            //设置zk监听的接口名称 - 类型 Array<string>, 必填, 在dubbo2.js@2.0.4+版本中不在使用这个参数
  service               //注入到dubbo容器的dubbo服务 - 类型Object, 在dubbo2.js@2.0.4+使用
})


//dubbo的代理服务
const demoSerivce = dubbo.proxService({
  //代理的服务接口 - string 必传
  dubboInterface: 'com.alibaba.dubbo.demo.DemoService',
  //服务接口的版本 - string 可选
  version: '1.0.0',
  //超时时间 number 可选
  timeout: 10
  //所属组 string 可选
  group: 'qianmi',
  //接口内的方法 - Array<Function> 必传
  methods: {
    //method name
    sayHello(name) {
      //仅做参数hessian化转换
      return [java.String(name)];
    },
    //method name
    getUserInfo() {
      //仅做参数hessian化转换
      return [
        java.combine('com.alibaba.dubbo.demo.UserRequest', {
          id: 1,
          name: 'nodejs',
          email: 'node@qianmi.com',
        }),
      ];
    },
  },
})
```

### connect dubbo directly

```typescript
import {DirectlyDubbo, java} from 'dubbo2.js';
import {
  DemoProvider,
  DemoProviderWrapper,
  IDemoProvider,
} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {UserRequest} from './providers/com/alibaba/dubbo/demo/UserRequest';

const dubbo = DirectlyDubbo.from({
  dubboAddress: 'localhost:20880',
  dubboVersion: '2.0.0',
  dubboInvokeTimeout: 10,
});

const demoService = dubbo.proxyService<IDemoProvider>({
  dubboInterface: 'com.alibaba.dubbo.demo.DemoProvider',
  methods: DemoProviderWrapper,
  version: '1.0.0',
});
```

## When dubbo was ready?

```javascript
const dubbo = Dubbo.from(/*...*/);

(async () => {
  await dubbo.ready();
  //TODO dubbo was ready
})();

//for example, in egg.js
app.beforeStart(async () => {
  await dubbo.ready();
  app.logger.info('dubbo was ready...');
});
```

## How to trace dubbo2.js runtime system info?

```javascript
const dubbo = Dubbo.from(/*...*/);

//通过subscribe
dubbo.subcribe({
  onTrace(msg: ITrace) {
    //logger msg
  },
});
```

You will get all runtim system info just like this.

```text
{ type: 'INFO', msg: 'dubbo:bootstrap version => 2.1.5' }
{ type: 'INFO', msg: 'connected to zkserver localhost:2181' }
{ type: 'INFO',
  msg: 'ServerAgent create socket-pool: 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'socket-pool: 172.19.6.203:20880 poolSize: 1' }
{ type: 'INFO',
  msg: 'new SocketWorker#1 |> 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'SocketWorker#1 <=connected=> 172.19.6.203:20880' }
{ type: 'INFO', msg: 'scheduler is ready' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'ERR',
  msg: Error: Can not be found any agents
    at Object.Scheduler._handleZkClientOnData [as onData] (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/scheduler.js:68:29)
    at EventEmitter.<anonymous> (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/zookeeper.js:275:30)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:118:7) }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 6
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 5
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'ERR',
  msg: Error: Can not be found any agents
    at Object.Scheduler._handleZkClientOnData [as onData] (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/scheduler.js:68:29)
    at EventEmitter.<anonymous> (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/zookeeper.js:275:30)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:118:7) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 4
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 3
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 2
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 1
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
{ type: 'ERR',
  msg:
   { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
{ type: 'ERR',
  msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 0
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'ERR',
  msg: Error: 172.19.6.203:20880's pool socket-worker had all closed. delete 172.19.6.203:20880
    at ServerAgent._clearClosedPool (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/server-agent.js:66:33)
    at Object.onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/server-agent.js:51:34)
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:97:34)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'ServerAgent create socket-pool: 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'socket-pool: 172.19.6.203:20880 poolSize: 1' }
{ type: 'INFO',
  msg: 'new SocketWorker#2 |> 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'SocketWorker#2 =connecting=> 172.19.6.203:20880' }
{ type: 'INFO',
  msg: 'SocketWorker#2 <=connected=> 172.19.6.203:20880' }
{ type: 'INFO', msg: 'scheduler is ready' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
{ type: 'INFO',
  msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
```

## middleware

抽象使用调用链路，并使用与 koa相同的 middleware机制。易于自定义拦截器，如：logger。

```js
//cost-time middleware
dubbo.use(async (ctx, next) => {
  const startTime = Date.now();
  await next();
  const endTime = Date.now();
  console.log('invoke cost time->', endTime - startTime);
});
```

## dubbo-invoker

dubbo 接口调用中，需设置某些动态参数，如：version, group, timeout, retry, 等。

参数在 consumer 调用时赋值。在此，抽象一个 dubbo-invoker 作为设置参数的 middleware，便于动态设置 runtime 参数。

```javascript
import {dubboInvoker, matcher} from 'dubbo-invoker';

//init
const dubbo = Dubbo.from(/*....*/);

const matchRuler = matcher
  //精确匹配接口
  .match('com.alibaba.demo.UserProvider', {
    version: '1.0.0',
    group: 'user',
  })
  //match thunk
  .match(ctx => {
    if (ctx.dubboInterface === 'com.alibaba.demo.ProductProvider') {
      ctx.version = '2.0.0';
      ctx.group = 'product-center';
      //通知dubboInvoker匹配成功
      return true;
    }
  })
  //正则匹配
  .match(/^com.alibaba.dubbo/, {
    version: '2.0.0',
    group: '',
  });

dubbo.use(dubboInvoke(matchRuler));
```

## Translator => Cool.

开发体验与用户体验同样重要。为此，dubbo2.js做了一些创新与实践。

为使 node 和 dubbo 之间的调用像 java 调用 dubbo 一样简单透明，其中设计并实现了 [translator](./packages/interpret-cli).

通过分析 java 的 jar 包中的 bytecode 提取 dubbo 调用的接口信息，自动生成 typescript 类型定义文件以及调用的代码。


**_职责_**

1.  翻译 Interface 代码,生成 node 端可调用代码;
2.  自动将参数转换为 hessian.js 能识别的对象;
3.  接口方法及参数类型提示;

[translator 详细介绍](./packages/interpret-cli)

## Performance

```text
❯ loadtest -t 20 -c 200 http://localhost:3000/dubbo -k
[Wed Jun 20 2018 15:10:16 GMT+0800 (CST)] INFO Requests: 0, requests per second: 0, mean latency: 0 ms
[Wed Jun 20 2018 15:10:21 GMT+0800 (CST)] INFO Requests: 37305, requests per second: 7484, mean latency: 26.9 ms
[Wed Jun 20 2018 15:10:26 GMT+0800 (CST)] INFO Requests: 79187, requests per second: 8371, mean latency: 23.9 ms
[Wed Jun 20 2018 15:10:31 GMT+0800 (CST)] INFO Requests: 120374, requests per second: 8247, mean latency: 24.2 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Target URL:          http://localhost:3000/dubbo
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Max time (s):        20
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Concurrency level:   200
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Agent:               keepalive
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Completed requests:  161828
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Total errors:        0
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Total time:          20.000902374000002 s
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Requests per second: 8091
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Mean latency:        24.7 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Percentage of the requests served within a certain time
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO   50%      22 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO   90%      30 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO   95%      34 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO   99%      49 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO  100%      134 ms (longest request)
```

## FAQ

```javascript
import {Dubbo} from 'dubbo2.js';
```

默认导入的 dubbo2.js 编译语言：es2017 （支持 node7.10+ ）。

低于node7.10的版本可使用：

```javascript
import {Dubbo} from 'dubbo2.js/es6';
```

## 怎么参与开发？

1.欢迎 pr

2.欢迎 issue

## 分享直戳

[talk](https://github.com/hufeng/iThink/tree/master/talk)
