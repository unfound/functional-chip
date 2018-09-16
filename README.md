# functional-chip

> 收集自己用过或者试验过的一些有用的代码片段或者算法实现，而不是简单地从别的地方fork后看都没看。人生路漫漫，自勉！

<!-- TOC -->

- [functional-chip](#functional-chip)
  - [数组扁平化](#数组扁平化)
  - [快速排序](#快速排序)
  - [二叉树](#二叉树)
  - [单例模式](#单例模式)
  - [函数节流与防抖](#函数节流与防抖)
    - [节流](#节流)
    - [去抖 || 消抖](#去抖--消抖)

<!-- /TOC -->

## 数组扁平化

将嵌套的多维数组变为一维数组，可能首先想到的是递归，但递归效率低且不优雅。可以如下实现 :

```javascript
function arrFlattening (arr) {
  return arr.toString().split(',')
}

var a = [1,[2,3,4,[5,[6],7],8],9,10]
console.log(arrFlattening(a))
// ['1','2','3','4','5','6','7','8','9','10']
```

## 快速排序

快速排序是一种效率比较高的排序方法，可用如下递归实现 :

```javascript
function quickSort (arr) {
  if (!Array.isArray(arr)) throw new Error('参数必须为数组！')
  if (arr.length <= 1) return arr
  var pivotIndex = Math.floor(arr.length / 2)
  var pivot = arr.splice(pivotIndex, 1)[0]
  var left = [], right = []
  arr.forEach(item => item > pivot ? right.push(item) : left.push(item))
  return quickSort(left).concat([pivot], quickSort(right))
}

var a = [1,8,9,6,5,3,4,7,10,2]
console.log(quickSort(a))
// [1,2,3,4,5,6,7,8,9,10]
```

当数据十分庞大的时候，递归实现容易造成栈溢出，此时可用非递归循环实现 :

```javascript
// 将数组以标志数划分成左右两个数组
function arrDivide (arr) {
  var middleIndex = Math.floor(arr.length / 2)
  // 尽量使得比较值接近平均值
  var pivot = Math.floor((arr[0] + arr[middleIndex] + arr[arr.length - 1]) / 3)
  var left = [], right = []
  arr.forEach(item => item > pivot ? right.push(item) : left.push(item))
  if (left.length !== 0 && right.length !== 0) {
    return [left, right]
  } else if (left.length === 0) {
    return [right]
  } else {
    return [left]
  }
}

function quickSort (arr) {
  if (!Array.isArray(arr)) throw new Error('参数必须为数组！')
  if (arr.length <= 1) return arr
  var list = [arr], length = arr.length
  // 快速排序分治到最后都变为长度为0或1的数组，将长度为0的数组排除，则list的长度应该与原数组长度相同
  while (list.length !== length) {
    for (var i = 0; i < list.length; i++) {
      if (list[i].length <= 1) continue
      list.splice(i, 1, ...arrDivide(list[i]))
    }
  }
  // 数组扁平化
  return list.toString().split(',')
}
```

> 上面算法实现中优化了标志数的选取，标志数越是接近中位数，效率越高。

上面的实现是基于切割数组进行排序的，空间复杂度会高点，但是代码比较易懂。也可以基于指针(下标)来实现空间复杂度会比较低 :

```javascript
function quickSort (arr, leftIndex, rightIndex) {
  if (leftIndex >= rightIndex) return arr
  var l = leftIndex, r = rightIndex, pivotIndex = leftIndex
  while (l < r) {
    while (arr[pivotIndex] <= arr[r] && pivotIndex < r) r--
    if (l >= r) break
    while (arr[pivotIndex] >= arr[l] && l < r) l++
    // 交换顺序不能错
    [arr[pivotIndex], arr[r], arr[l]] = [arr[r], arr[l], arr[pivotIndex]]
    pivotIndex = l
  }
  quickSort(arr, leftIndex, pivotIndex - 1)
  quickSort(arr, pivotIndex + 1, rightIndex)
}
```

非递归实现 :

```javascript
function quickSort (arr) {
  // 用list数组来存储左右指针
  var list = [[0, arr.length - 1]]
  while (list.length > 0) {
    var now = list.pop()
    if (now[0] >= now[1]) continue
    var l = now[0], r = now[1], pivotIndex = now[0]
    while (l < r) {
      while (arr[pivotIndex] <= arr[r] && pivotIndex < r) r--
      if (l >= r) break
      while (arr[pivotIndex] >= arr[l] && l < r) l++
      // 交换顺序不能错
      [arr[pivotIndex], arr[r], arr[l]] = [arr[r], arr[l], arr[pivotIndex]]
      pivotIndex = l
    }
    list.push([now[0], pivotIndex - 1])
    list.push([pivotIndex + 1, now[1]])
  }
}
```

## 二叉树

二叉树是一种常用的数据结构，下面列出二叉搜索树的实现，先是树的生成 :

```javascript
function Node (data, left, right) {
  this.data = data // 节点数据
  this.left = left // 左节点
  this.right = right // 右节点
}
// 节点数据获取
Node.prototype.show = function () {
  return this.data
}
// 二叉搜索树构造函数
function BST () {
  this.root = null // 根节点
}
// 节点插入，也就是树的生成
BST.prototype.insert = function (data) {
  var node = new Node(data, null, null)
  if (this.root === null) {
    this.root = node
    return
  }
  var current = this.root, parent = null
  while (current) {
    parent = current
    if (data < current.data) {
      current = current.left
      if (current === null) {
        parent.left = node
        break
      }
    } else {
      current = current.right
      if (current === null) {
        parent.right = node
        break
      }
    }
  }
}
```

中序遍历的递归实现 :

```javascript
BST.prototype.centerOrder = function (node) {
  if (node === null) return []
  return this.centerOrder(node.left).concat([node.show()], this.centerOrder(node.right))
}
```

非递归实现 :

```javascript
BST.prototype.centerOrderArr = function () {
  var list = [], dataList = [], current = this.root
  while (current !== null || list.length !== 0) {
    while (current !== null) {
      list.push(current)
      current = current.left
    }
    current = list.pop()
    dataList.push(current.show())
    current = current.right
  }
  return dataList
}
```

## 单例模式

单例模式是为了保证生成的实例对象在整个系统中仅有一个，可以用来节省内存以及简化复杂环境下的配置管理

```javascript
// 闭包实现
var single = (() => {
  var unique

  function getInstance () {
    if (unique === undefined) {
      unique = new Construct()
    }
    return unique
  }

  function Construct () {/* 单例构造函数 */}

  return {
    // 获取单例的方法
    getInstance: getInstance
  }
})()

// 构造函数实现
function Single () {
  if (Single.unique !== undefined) return Single.unique
  // 类似java中的静态变量
  Single.unique = this
}

// 对象字面量实现
var single = {/* 对象属性 */}
```

## 函数节流与防抖

### 节流

函数节流指的是指定一段时间内，函数仅执行一次，通常用来优化频繁的事件触发。如，scroll，resize等。

> 最简单的就是在执行函数内套一层 **requestAnimationFrame**。对于频繁的动画效果具有非常好的优化效果

```javascript
window.onscroll = () => {
  requestAnimationFrame(() => {
    //事件处理代码
  })
}
```

> 上面代码虽然简单，但是 **requestAnimationFrame** 是根据浏览器的刷新频率(通常是16ms)间隔来节流，不能自己来定间隔，但16ms的间隔很多时候还是过于频繁了，且 **requestAnimationFrame** 严格来讲并没有对函数运算做节流，只是把动画效果合并，提升了页面的渲染速度。所以，我们还是需要一个节流函数来对有着大量运算的函数进行节流。

```javascript
/**
 * 函数节流的简单实现
 * @param {Function} func 需要节流的函数
 * @param {Number} delay 延时单位ms
 */

function throttle (func, delay) {
  let last = 0
  return function () {
    const current = +new Date()
    if (current - last > delay) {
      func.apply(this, arguments)
      last = current
    }
  }
}

/**
 * underscore里的节流函数源码修改
 * @param {Function} func 需要节流的函数
 * @param {Number} delay 延时单位ms
 * @param {{immediate: Boolean, trailing: Boolean}} opts 配置项 
 *   @param {Boolean} opts.immediate 是否一触发就立即执行，默认是
 *   @param {Boolean} opts.trailing 最后是否再执行一次，默认否
 * 
 */
function throttle (func, delay, opts = {}) {
  var ctx, args, res, timer = null, last = 0
  opts.immediate === undefined ? opts.immediate = true : false
  // 拖尾
  function later () {
    // 在拖尾之后初始化last以便于下个循环开始时，
    last = opts.immediate ? +new Date() : 0
    timer = null
    res = func.apply(ctx, args)
    ctx = args = null
  }
  return function () {
    var now = +new Date()
    if (!last && !opts.immediate) last = now
    // 时间差
    var remaining = delay - (now - last)
    ctx = this
    args = arguments
    if (remaining <= 0 || remaining > delay) {
      if (timer) {
        clearTimeout(timer)
        timer = null
      }
      last = now
      res = func.apply(ctx, args)
      if (!timer) ctx = args = null
    } else if (!timer && opts.trailing) {
      timer = setTimeout(later, remaining)
    }
    return res
  }
}
```

### 去抖 || 消抖

去抖或者说消抖指的是，一个函数在某个条件下必须延时规定时间再执行，如果该条件在延时时间内再次触发则时间重置，在延时规定时间再执行。语文不行，讲起来有点绕口了==举个栗子，比如说电梯门吧，出于安全考虑规定没人通过门3s后关闭，如果到了2s有人进来了，那么需要再过3s没人进来才会关闭，总不能到第3s就马上关了，那第3s进来的人可真惨。当然如果每过2s都有人进来的话，建议你还是爬楼梯吧==

```javascript
/**
 * 函数去抖的简单实现
 * @param {Function} func 需要节流的函数
 * @param {Number} delay 延时单位ms
 */
function debounce (func, delay) {
  clearTimeout(func.fid)
  func.fid = setTimeout(() => func.call(this), delay)
}

function debounce (func, delay) {
  var timer = null;
  return function () {
    clearTimeout(timer);
    timer = setTimeout(() => func.apply(this, arguments), delay)
  }
}

/**
 * underscore里的防抖函数源码修改
 * @param {Function} func 需要防抖的函数
 * @param {Number} delay 延时单位ms
 * @param {Boolean} immediate 是否第一次调用立即执行，默认是
 */
function debouce (func, delay, immediate = false) {
  var timer, args, ctx, timestamp, res
  var later = function () {
    // 如果未达到指定间隔，重复调用函数会不断刷新timestamp
    var last = +new Date() - timestamp
    if (last < delay && last > 0) {
      // 未达到指定间隔，重复调用，则重新起一个定时器
      timer = setTimeout(later, delay - last)
    } else {
      timer = null
      if (!immediate) {
        res = func.apply(ctx, args)
        ctx = args = null
      }
    }
  }

  return function () {
    ctx = this
    args = arguments
    timestamp = +new Date()
    // 没有定时器且immediate为true时立即调用
    var callNow = immediate && !timer
    // 没有定时器时，设置一个定时器
    if (!timer) timer = setTimeout(later, delay)
    if (callNow) {
      res = func.apply(ctx, args)
      ctx = args = null
    }
    return res
  }
}
```