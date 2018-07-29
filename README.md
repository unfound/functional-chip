# functional-chip

> 收集自己用过或者试验过的一些有用的代码片段或者算法实现，而不是简单地从别的地方fork后看都没看。人生路漫漫，自勉！

## 数组拍扁
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

上面的实现是基于切割数组进行排序的，空间复杂度会高点，当时代码比较易懂。也可以基于指针(下标)来实现空间复杂度会比较低 :

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
