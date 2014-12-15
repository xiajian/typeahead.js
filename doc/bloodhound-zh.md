Bloodhound
=========

Bloodhound是typeahead.js的推荐引擎，健壮且灵活，而且还提供诸如预取，智能缓存，快速查找，远程数据回填。下面，从特性和使用方面分别介绍，使用则从API、选项、预取等方面介绍。

其特性如下: 

* 可使用硬编码数据
* 初始化是预取数据，从而减少推荐时延
* 智能使用本地存储从而减少网络请求
* 从远程源中回填建议项
* 对远程请求的速率限制并缓存网络请求，从而减轻负载

具体的使用，按API，选项，数据集，定制事件和 Look and Feel介绍。

## API

* `new Bloodhound(options)`是构造函数，其接受哈希选项作为唯一的参数。

```javascript
var engine = new Bloodhound({
  name: 'animals',
  local: [{ val: 'dog' }, { val: 'pig' }, { val: 'moose' }], // 本地数据源
  remote: 'http://example.com/animals?q=%QUERY',             // 远程数据源
  datumTokenizer: function(d) {
    return Bloodhound.tokenizers.whitespace(d.val);
  },
  queryTokenizer: Bloodhound.tokenizers.whitespace
});
```

* `Bloodhound#initialize(reinitialize)`启动了推荐引擎。其启动过程包括处理有`local`以及`prefetch`提供的数据。初始化完成前，其他方法均不起作用; 初始化完成后，返回一个 [jQuery promise] 。

```javascript
var promise = engine.initialize();

promise
.done(function() { console.log('success!'); })
.fail(function() { console.log('err!'); });
```

After the initial call of `initialize`, how subsequent invocations of the method
behave depends on the `reinitialize` argument. If `reinitialize` is false, the
method will not execute the initialization logic and will just return the same 
jQuery promise returned by the initial invocation. If `reinitialize` is truthy,
the method will behave as if it were being called for the first time.

在`initialize`调用之后，后续调用方法的行为依赖于`reinitialize`参数。如果`reinitialize`为false，函数不执行初始化逻辑，并仅仅返回和最初调用相同的jQuery promise。如果`reinitialize`为真，初始化就重新执行。

```javascript
var promise1 = engine.initialize();
var promise2 = engine.initialize();
var promise3 = engine.initialize(true);

promise1 === promise2; // 相等性判断
promise3 !== promise1 && promise3 !== promise2;
```

* `Bloodhound#add(datums)`获取一个数据参数datums，其中的数据将被添加到搜索索引中，从而增强推荐引擎。

```javascript
engine.add([{ val: 'one' }, { val: 'two' }]);
```

* `Bloodhound#clear()` 从搜索引擎中移除所有推荐。

```javascript
engine.clear();
```

* `Bloodhound#clearPrefetchCache()` 如果使用了`prefetch`选项，数据将会缓存到本地存储中，从而减少不必要的网络请求。`clearPrefetchCache`提供了可编程移除缓存的方法。

```javascript
engine.clearPrefetchCache();
```

* `Bloodhound#clearRemoteCache()` 如果使用了`remote`选项，Bloodhound将会缓存最近10条响应，从而提供最佳的用户体验。`clearRemoteCache`提供编程清除缓存的方法。

```javascript
engine.clearRemoteCache();
```

* `Bloodhound.noConflict()` 返回Bloodhound构造器的引用，并返回`window.Bloodhound`的先前的值。可以用来避免命名冲突。

```javascript
var Dachshund = Bloodhound.noConflict();
```

<!-- section links -->

[jQuery promise]: http://api.jquery.com/Types/#Promise

* `Bloodhound#get(query, cb)` 为`query`计算一组建议。`cb`为之前提到datums。`cb`总是与在客户端可用的建议一起同步调用。如果客户端的不够，将会考虑`remote`。`cb`也可以异步的方式，混合着客户端和`remote`端数据进行调用。注： 例子中显示，`cb`为函数。

```javascript
bloodhound.get(myQuery, function(suggestions) {
  suggestions.each(function(suggestion) { console.log(suggestion); });
});
```

### Options


当实例化一个Bloodhund推荐引擎时，存在如下的选项可配置: 

* `datumTokenizer` – 将datum转换为字符串token的数组的函数，其签名为`(datum)`。**必须**
* `queryTokenizer` – 将查询转换为字符串token的数组的函数，其签名为`(query)`。**必须**
* `limit` – `Bloodhound#get`方法返回的最大的建议数目. 如果不够，就用远程的数据回填。默认为5个。
* `dupDetector` – 如果设置该选项，其值为带有`(remoteMatch, localMatch)`选项的冗余检测函数，即数据中存在冗余，则返回为true，否则返回为false。如果不设置，则不执行冗余检测。
* `sorter` – 用来为给定查询所匹配的数据进行排序。
* `local` – datums数组或返回datums数组的函数。
* `prefetch` – 可以是包含datums数组的JSON文件的URL，更多信息，参考[prefetch]哈希选项。
* `remote` – 当`local`和`prefetch`提供的数据不充分，或需要更多的可配置性时，用来从远程获取建议的URL。

<!-- section links -->

[compare function]: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort>

### Prefetch

在初始化过程中，预取并处理数据。如果浏览器支持本地存储，处理数据将被缓存，从而避免后续页面加载中额外的网络请求。

**警告** 尽管小数据集可以不用预取缓存，并且预取不意味着包含整个数据集。相反的，需将其看作推荐的一级缓存。如果不谨记在心，可能会触及本地缓存限制。

当配置`prefetch`时，如下选项亦可用:

* `url` – 链接到包含datums数组的JSON文件的URL。**必须**
* `cacheKey` – 存储在本地的数据的key。默认为`url`的值
* `ttl` – 预取的数据缓存在本地存储中的时间(毫秒)，默认为`86400000` (1天)
* `thumbprint` – 用作预取数据的数据指纹。如果和本地缓存的数据不匹配，数据将会被重新获取
* `filter` – 将响应体转换成datums的数组的函数，其签名为`filter(parsedResponse)`，返回值为datums数组。
* `ajax` – 传递给`jQuery.ajax`的ajax设置

<!-- section links -->

[local storage limits]: http://stackoverflow.com/a/2989317
[ajax settings object]:http://api.jquery.com/jQuery.ajax/#jQuery-ajax-settings

### Remote

远程数据仅在`local`和`prefetch`提供的数据不充分时，才会使用。为了避免到远程端过于繁琐的请求数，请求被限速了。

当配置`remote`时，如下的选项是可用的: 

* `url` – 当`local`和`prefetch`的数据不足时，发出请求的URL。**必须**
* `wildcard` - `url`中的模式，当发出请求时，将会被替换为用户的查询。默认为`%QUERY`
* `replace` – 用来覆盖请求URL的函数, 其方法签名为`replace(url, query)`，返回值为一合法的URL。如果设置，将不对URL执行任何替换？
* `rateLimitBy` – 该方法用来实时显示网络请求数目。其值为`debounce`或`throttle`，默认为`debounce`。
* `rateLimitWait` – `rateLimitBy`所使用的时间间隔(毫秒)，默认为`300`
* `filter` – 将响应体转换为数组datums的函数，其方法签名为`filter(parsedResponse)`，返回值为datums数组
* `ajax` – 传给`jQuery.ajax`ajax设置对象

<!-- section links -->

[ajax settings object]: http://api.jquery.com/jQuery.ajax/#jQuery-ajax-settings

### Datums

Datums are JavaScript objects that hydrate the pool of possible suggestions.
Bloodhound doesn't expect datums to contain any specific properties as any
operations performed on datums are done using functions defined by the user i.e.
`datumTokenizer`, `dupDetector`, and `sorter`.

### Tokens

The algorithm used by bloodhounds for providing suggestions for a given query 
is token-based. When `Bloodhound#get` is called, it tokenizes `query` using 
`queryTokenizer` and then invokes `cb` with all of the datums that contain those 
tokens.

For a quick example, if a datum was tokenized into the following set of 
tokens...

```javascript
['typeahead.js', 'typeahead', 'autocomplete', 'javascript'];
```

...it would be a valid match for queries such as:

* `typehead`
* `typehead.js`
* `autoco`
* `java type`

