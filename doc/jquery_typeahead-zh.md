typeahead
========

typeahead.js的UI组件是一个jQuery插件，其负责渲染建议并处理DOM交互。其特性如下: 

* 在用户输入时，显示建议
* 将顶层推荐显示为hint(例如，背景文字)
* 支持可定制的模板
* 同RTL语言和输入编辑器工作良好
* 高亮匹配的查询
* 触发定制事件

## 规范

为了最大程度的利用用户现有的关于typeahead.js的知识，typeahead.js UI的行为是仿照谷歌的搜索框。下面的伪代码，介绍UI界面如何处理相关事件的。

预输入需要考虑的事件有: 聚焦是失焦，值的改变，方向键，Tab键，Esc键

**输入控件聚焦**

```
activate typeahead  //激活typeahead
```

**输入控件失去焦点**

```
deactivate typeahead //失效typeahead
close dropdown menu  //关闭下拉菜单
remove hint          //移除hint
clear suggestions from dropdown menu //清理下拉菜单中的推荐
```

**输入控件中的值的改变**

```
IF query satisfies minLength requirement THEN  //查询满足最小长度需求
  request suggestions for new query            //为新的查询请求推荐

  IF suggestions are available THEN  
    render suggestions in dropdown menu
    open dropdown menu 
    update hint
  ELSE
    close dropdown menu 
    clear suggestions from dropdown menu
    remove hint
  ENDIF
ELSE
  close dropdown menu 
  clear suggestions from dropdown menu
  remove hint
ENDIF
```

**按下Up键**

```
IF dropdown menu is open THEN
  move dropdown menu cursor up 1 suggestion
ELSE
  request suggestions for current query

  IF suggestions are available THEN
    render suggestions in dropdown menu
    open dropdown menu 
    update hint
  ENDIF
ENDIF
```

**按下Down键**

```
IF dropdown menu is open THEN
  move dropdown menu cursor down 1 suggestion
ELSE
  request suggestions for current query

  IF suggestions are available THEN
    render suggestions in dropdown menu
    open dropdown menu 
    update hint
  ENDIF
ENDIF
```

**左箭头按下**

```
IF detected query language direction is right-to-left THEN
  IF hint is being shown THEN
    IF text cursor is at end of query THEN
      autocomplete query to hint
    ENDIF
  ENDIF
ENDIF
```

**右箭头按下**

```
IF detected query language direction is left-to-right THEN
  IF hint is being shown THEN
    IF text cursor is at the end of the query THEN
      autocomplete query to hint
    ENDIF
  ENDIF
ENDIF
```

**按Tab键**

```
IF dropdown menu cursor is on suggestion THEN
  close dropdown menu
  update query to display key of suggestion
  remove hint
ELSIF hint is being shown THEN
  autocomplete query to hint
ENDIF
```

**Enter按键**

```
IF dropdown menu cursor is on suggestion THEN
  close dropdown menu
  update query to display key of suggestion
  remove hint
  prevent default browser action e.g. form submit
ENDIF
```

**Esc按键**

```
close dropdown menu
remove hint
```

**点击推荐项**

```
update query to display key of suggestion
close dropdown menu
remove hint
```

### API

* `jQuery#typeahead(options, [\*datasets])`

Turns any `input[type="text"]` element into a typeahead. `options` is an 
options hash that's used to configure the typeahead to your liking. Refer to 
[Options](#options) for more info regarding the available configs. Subsequent 
arguments (`*datasets`), are individual option hashes for datasets. For more 
details regarding datasets, refer to [Datasets](#datasets).

将任何`input[type="text"]`元素转换成typehead。`options`参数hash将typeahead配置为你所喜欢的，`*datasets`参数是配置数据集的独立选项hash。更多关于选项的信息参考下文。

```javascript
$('.typeahead').typeahead({
  minLength: 3,
  highlight: true,
},
{
  name: 'my-dataset',
  source: mySource
});
```

* `jQuery#typeahead('destroy')` 移除typeahead功能，并将`input`元素的状态重置为原始状态。

```javascript
$('.typeahead').typeahead('destroy');
```

* `jQuery#typeahead('open')` 打开typeahead下拉菜单。 注意，打开菜单不意味着菜单可见。仅当其打开并存在内容时，菜单才可见。

```javascript
$('.typeahead').typeahead('open');
```

* `jQuery#typeahead('close')` 关闭typeahead的下拉菜单。

```javascript
$('.typeahead').typeahead('close');
```

* `jQuery#typeahead('val')` 返回typeahead的当前值，该值为用户输入到`input`元素中的文本。

```javascript
var myVal = $('.typeahead').typeahead('val');
```

* `jQuery#typeahead('val', val)` 设置typeahead的值，要来替代`jQuery#val`函数。

```javascript
$('.typeahead').typeahead('val', myVal);
```

* `jQuery.fn.typeahead.noConflict()` 返回typeahead插件的引用，并将`jQuery.fn.typeahead`重置为先前值。这可以用来避免命名冲突。

```javascript
var typeahead = jQuery.fn.typeahead.noConflict();
jQuery.fn._typeahead = typeahead;
```

### 选项(Options)

当初始化typeahead时，存在如下的可配置的选项:

* `highlight` – 设置为`true`时，当建议渲染时，在文本节点中，匹配查询模式的文字将被带有`tt-highlight` class的`strong`元素包裹。默认设置为`false`。
* `hint` – 设置为`false`时，typeahead 将不会显示hint。默认为`true`.
* `minLength` – 推荐引擎开始渲染所需要的最小字符。默认为 `1`.

### 数据集(Datasets)

`typeahead`可以由一个或多个数据集组成。但用户修改typeahead的值时，每个数据集都会尝试渲染为新的查询渲染值。

大多数情况下，一个数据集足够了。只有在需要在下拉菜单中，以某些分类关系分组渲染推荐时，才需要使用多个数据源。例如，在`twitter.com`中，搜索预输入将结果分组为相关搜索，趋势，账户 - 这就需要使用多个数据集。

数据集可以通过如下的选项进行配置: 

* `source` – 推荐的数据源支持。值为带有`(query, cb)`签名的函数。该函数将会计为`query`计算推荐集，然后以计算的推荐集调用`cb`。函数`cb`的调用可以是同步的，也可以是异步的。Bloodhound推荐引擎可在这里使用，更多参考[Bloodhound Integration]。**必须**

* `name` – 数据集的名字。该名字可以被追加到`tt-dataset-`，从而形成包含DOM元素的类名。只能由下划线、-，字母和数字组成。默认为随机数。

* `displayKey` – 对于一个给定的推荐对象，决定其的字符串表示，并将会在某个输入控件选择后使用。其值可以是关键字符串，或者是将推荐对象转换为string的函数。默认为`value`。
* `templates` – 渲染数据集使用的哈希模板。注意：预编译的模板是将javascript对象作为第一参数，并返回为HTML字符串。

  * `empty` – 当给定查询推荐数为0时，渲染`empty`中的内容。`empty`的值可以是HTML字符串或预编译模板。如果是预编译模板，其内容中将包含`query`。 

  * `footer` – 数据集底部渲染的内容，可为HTML字符串或预编译模板。如果是预编译模板，其中包含`query`和`isEmpty`。

  * `header` – 数据集头部渲染的内容，可为HTML字符串或预编译模板。如果是预编译模板，其中包含`query`和 `isEmpty`。

  * `suggestion` – 用来渲染单个推荐。其值必须是预编译模板。其中包含关联的建议对象。默认为将`displayKey`包装在`p`标签中：`<p>{{value}}</p>`

### Custom Events

typeahead组件触发了如下的定制的事件: 

* `typeahead:opened` – 当typeahead的下拉菜单打开时触发。
* `typeahead:closed` – 当typeahead的下拉菜单关闭时触发。
* `typeahead:cursorchanged` – 但下拉菜单的光标移动到另一个推荐时，触发事件。事件处理器将接受三个参数： jQuery事件对象、推荐对象以及推荐对象所属的数据集名。
* `typeahead:selected` – 当下拉菜单被渲染时触发。事件处理器接受三个参数： jQuery事件对象，推荐对象以及推荐对象所属的数据集名。
* `typeahead:autocompleted` – 当查询自动补全时触发。自动补全意味着将`hint`改变为查询，事件器将调用3个参数: jQuery事件对象、推荐对象以及推荐对象所属的数据集名。 

All custom events are triggered on the element initialized as a typeahead.

所有的定制事件将会在元素初始化为typeahead时触发。

### Look and Feel

下面是虚构的mustache模板，用来描述typeahead下拉菜单DOM元素结构。 注意，`header`, `footer`, `suggestion`以及 `empty`都是有datase提供的模板。

```html
<span class="tt-dropdown-menu">
  {{#datasets}}
    <div class="tt-dataset-{{name}}">
      {{{header}}}
      <span class="tt-suggestions">
        {{#suggestions}}
          <div class="tt-suggestion">{{{suggestion}}}</div>
        {{/suggestions}}
        {{^suggestions}}
          {{{empty}}}
        {{/suggestions}}
      </span>
      {{{footer}}}
    </div>
  {{/datasets}}
</span>
```

当用户在`.tt-suggestion`的a链接上移动鼠标或键盘时，将会追加样式`tt-cursor`。可使用该样式类标识位于光标下的推荐。

## Bloodhound Integration

由于数据集期望`source`属性是一个函数，所以，不能直接将Bloodhound推荐引擎传递进来。相反的，需要使用推荐引擎的typeahead适配器`ttAdapter`。

```javascript
var engine = new Bloodhound({ /* options */ });

engine.initialize();

$('.typeahead').typeahead(null, {
  displayKey: myDisplayKey // if not set, will default to 'value',
  source: engine.ttAdapter()
});
```

