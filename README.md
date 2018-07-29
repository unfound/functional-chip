# functional-chip

> 收集自己用过或者试验过的一些有用的代码片段或者算法实现，而不是简单地从别的地方fork后看都没看。人生路漫漫，自勉！

## 数组拍扁

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