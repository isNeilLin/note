# RxJS-基本概念
Rx(ReactiveX)是结合了**观察者模式**、**迭代器模式**和**使用集合**的**函数式编程**。
RxJS中用来解决异步事件管理的基本概念是：
- Observable(被观察者): 一个可调用的未来值或事件的集合
- Observer(观察者): 一个回调函数的集合，它知道如何去监听由Observable提供的值
- Subscription(订阅): 表示Observable的执行，主要用于取消Observable的执行
- Operators(操作符): 采用函数式编程风格的纯函数
- Subject(主体): 相当于EventEmitter，并且是将值或事件多路推送给多个Observer的唯一方式
- Schedulers(调度器): 用来控制并发并且是中央集权的调度员，允许在发生计算时进行协调

## Observable(被观察者)

#### 拉取(pull)&推送(push)

拉取：在拉取体系中，由消费者决定何时从数据生产者那里接收数据。每个javascript函数都是拉取体系，函数是数据的生产者，调用该函数从函数调用中取出一个返回值对该函数进行消费。ES6引入了`generator`函数和`iterators(function*)`，这是另外一种类型的拉取体系，调用`iterator.next()`的代码是消费者，他从`iterator`(生产者)那里取出多个值。

推送：在推送体系中，由生产者决定何时把数据发送给消费者，消费者本身不知道何时会接收到数据。`Promise`是最常见的推送体系类型。Promise(生产者)将一个解析过的值传递给已注册过的回调函数(消费者)，但不同于函数的是由Promise决定何时把数据推送给回调函数。

RxJS引入了Observable，一个新的推送体系，Observable是多个值的生产者，并将值推送给观察者(消费者)

```javascript
var observable = Rx.Observable.create(function(observer){
	observer.next(1);
	observer.next(2);
	observer.next(3);
	setTimeout(function(){
		observer.next(4);
		observer.complete();
	},1000)
})
```
要调用observable并看到这些值，需要*订阅*observable
```javascript
console.log('before subscribe')
observable.subscribe({
	next: x => {
		console.log(x)
	},
	error: err => {
		console.error(err)
	},
	complete: () => {
		console.log('done')
	}
})
console.log('after subscribe')
```
执行结果：
```
before subscribe
1
2
3
after subscribe
4
done
```

- **Function** 调用时会同步地返回一个单一值。
- **Generator** 调用时会同步地返回零到(有可能的)无限多个值。
- **Promise** 是最终可能(或可能不)返回单个值的运算。
- **Observable** 它可以从它被调用的时刻起同步或异步地返回零到(有可能的)无限多个值。

#### Observables 作为函数的泛化

Observables像是没有参数，但可以泛化为多个值的函数。
订阅Observables类似于调用函数。
Observables传递值可以是同步的，也可以是异步的。

•	function.call() 意思是 "同步地给我一个值"
•	observable.subscribe() 意思是 "给我任意数量的值，无论是同步还是异步"

#### Observable的核心关注点：

- 创建
- 订阅
- 执行
- 清理

**创建Observables**

`Rx.Observable.create`是`Observable`构造函数的别名，它接收一个参数：`subscribe`函数。

示例：

```javascript
var observable = Rx.Observable.create(function subscribe(observer){
	observer.next('hi');
})
```

> Observable可以使用Rx.Observable.create创建，但通常我们使用所谓的**创建操作符**，像`of`,`from`,`interval`等等。

**订阅Observables**

observable可以订阅：`observable.subscribe()`
`subscribe`调用在同一个`Observable`的多个观察者之间是不共享的。`Observable.create(function subscribe(observer){...})`中的`subscribe`函数只服务于给定的观察者，对`observable.subscribe()`的每次调用都会触发针对给定观察者的独立设置。

> 订阅`Observable`像是调用函数, 并提供接收数据的回调函数。

**执行Observable**

`Observable.create(function subscribe(observer){...})`中的`...`表示Observable执行，它是惰性运算，只有在每个观察者订阅之后才会执行。随着时间的推移，执行会以同步或异步的方式产生多个值。

Observable执行可以传递三种类型的值：
- `next`通知： 发送一个值
- `error`通知： 发送一个JavaScript错误或异常
- `complete`通知：不再发送任何值

在`subscribe`中可以使用`try/catch`代码块包裹任意代码，如果捕获到异常，发送`error`通知

```javascript
var observable = Rx.Observable.create(function subscribe(observer){
	try{
		observer.next(1)
		observer.next(2)
		observer.complete()
	}catch(e){
		observer.error(e) // 如果捕获到异常会发送一个错误
	}
})
```

**清理Observable执行**

当调用了`observable.subscribe`，观察者会被附加到新创建的observable执行中。这个调用还返回一个对象：`Subscription`(订阅)

```javascript
var subscription = observable.subscribe({
	next: x => {console.log(x)},
	error: err => {console.log(err)},
	complete: () => {console.log('done')}
})
subscription.unsubscribe();
```
Subscription表示进行中的执行，调用`subscription.unsubscribe()`可以取消进行中的执行

## Observer（观察者）

观察者只是有三个回调函数的对象，每个回调函数对应一种`Observable`发送的通知类型。要使用观察者，需要把它提供给`Observable`的`subscribe`方法

RxJS中的观察者也可能是部分的，如果没有提供某个回调函数，`Observable`的执行也会正常运行。只是某些同志类型会被忽略。

当订阅Observable时，如果只提供了一个回调函数作为参数，而没有将其附加到观察者对象上

```
var subscription = observable.subscribe(x=>{
	console.log(x)
})
```

在`observable.subscribe`内部，它会创建一个观察者对象并使用第一个回调函数参数作为 next 的处理方法。所有三种类型的回调函数都可以直接作为参数来提供

```
var subscription = observable.subscribe(
	x => {console.log(x)},
	err => {console.log(err)},
	() => {console.log('done')}
)
```

## Subscription(订阅)

subscription表示可以清理资源的对象，通常是observable的执行。Subscription 基本上只有一个 `unsubscribe()` 函数，这个函数用来释放资源或去取消 Observable 执行。
subscription还可以合在一起，这样一个subscription调用`unsubscribe()`方法可能会有多个subscription取消订阅

```javascript
var observable1 = Rx.Observable.interval(400);
var observable2 = Rx.Observable.interval(300);

var subscription1 = observable1.subscribe(
	x=>{console.log(x)}
)

var subscription2 = observable2.subscribe(
	x=>{console.log(x)}
)

subscription1.add(subscription2);

setTimeout(function(){
	// subscription1 和 subscription2 都会取消订阅
	subscription1.unsubscribe()
},1000)
```


