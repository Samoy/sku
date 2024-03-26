# 一文说清SKU算法的实现

## 什么是SKU

SKU（Stock Keeping Unit）是库存管理中的基本概念，指的是库存进出计量的单位，也是产品库存控制的最小可用单位。
SKU是商业运营中对商品进行管理和追踪的一种独特标识符，通常是由一串数字或字母数字组合而成的代码。
对于不同的商品，即使颜色、尺寸、款式等属性有所差异，也会被赋予不同的SKU号，这样就能在库存系统中精确区分每一种具体的产品变体。

## 业务场景

当我们在选择商品时，通常会根据用户每一次的选择，来找出其余属性是否能被选择，如果无法被选择，则需要将其置灰，避免用户选择无效的SKU。也就是如下效果：
![演示示例](https://www.samoy.site/sku/sku-0.gif)

## Demo地址
[点击这里](https://www.samoy.site/sku)

## 实现思路

最简单的实现思路就是遍历所有可选的SKU，然后去一一进行匹配，如果匹配成功，则将当前的属性值设置为可选状态，否则为置灰状态。
但是，这种方式效率较低，时间复杂度较高，可能会导致页面渲染缓慢。因此我们需要另寻他法。

## 大佬经验

1. [使用无向图](https://juejin.cn/post/6844904196349640718)  
   根据这篇文章下的评论，使用无向图会有Bug，无法解决边公用的问题。
2. [使用质数](https://juejin.cn/post/7166229947806122015)  
   另一种解决方案是使用质数。质数可以避免边公用的问题。在这篇文章下面的评论，有人提议使用位运算，但是没有给出具体的实现。
   因此我们就尝试使用位运算来解决此问题。接下来我们来了解一下位运算。

## 位运算

在这里使用位运算，我们需要了解两个关键概念：左移、按位与。

1. 左移：**左移操作符 (<<)** 将第一个操作数向左移动指定位数，左边超出的位数将会被清除，右边将会补零。如`a << b`，即`a`
   向左移动`b`
   位。在十进制中的数学实现为`a = a * 2^b`。例如：`1 << 0 = 1`, `1 << 1 = 2`，`1 << 2 = 4`。
2. 按位与：**按位与操作符 (&)** 将两个操作数转换为32位（在32位计算机上）二进制后，按位进行与运算，结果为1的位，则为1，否则为0。
   例如：`1`为`00000000000000000000000000000001`，`2`为`000000000000000000000000000010`, `1&2`的结果是
   `00000000000000000000000000000000`， 即十进制的`0`。

## 规律

首先我们观察一下：  
1的二进制为`00000000000000000000000000000001`，  
2的二进制为`00000000000000000000000000000010`，  
4的二进制为`00000000000000000000000000000100`，  
8的二进制为`00000000000000000000000000001000`  。  
那么我们可以发现，如果一个数是2的幂次方，那么它的二进制只有一位为1，其余位均为0。而`1+2+4+8`的结果是`15`，
转换为二进制为`00000000000000000000000000001111`。

* 我们将`15`与`1`进行按位与运算，结果为`00000000000000000000000000000001`，十进制为`1`。
* 我们将`15`与`2`进行按位与运算，结果为`00000000000000000000000000000010`，十进制为`2`。
* 我们将`15`与`4`进行按位与运算，结果为`00000000000000000000000000000100`，十进制为`4`。
* 我们将`15`与`8`进行按位与运算，结果为`00000000000000000000000000001000`，十进制为`8`。
* 我们将`15`与`1+2=3`进行按位与运算，结果为`0000000000000000000000000000000011`，十进制为`3`。
* 我们将`15`与`1+2+4=7`进行按位与运算，结果为`0000000000000000000000000000000111`，十进制为`7`。
* 我们将`15`与`1+4=5`进行按位与运算，结果为`0000000000000000000000000000000101`，十进制为`5`。

现在我们已经发现了规律：`1 + 2 + 4 + 8 = 15`分别与其**某几个加数之和**进行**按位与**运算，结果是**这几个加数之和**
。用数学公式表示为
`(1<<0 + 1<<1 + ... + 1<<n ) & 任意某几个加数之和 = 这几个加数的和`。

## 结论

这样，我们就可以得出结论：
首先，`(1<<k)` 表示将1向左移动 k 位，其结果是一个二进制数，仅最高位（第 k+1 位）为1，其余各位均为0。例如：

* `(1<<1)` 等于`0b10`，即十进制的`2`。
* `(1<<2)` 等于`0b100`，即十进制的`4`。
* 以此类推...
  现在考虑求和表达式`(1<<0)+(1<<1)+...+(1<<n)`，这个求和的结果实际上是一个二进制数，其中第 1 到第 n+1 位分别为1，其余位为0。
  当我们对该结果与其中任意一个加数 `(1<<k)` 进行按位与操作（`&`）时：
* 对于任何不是第`k+1`位的位置，两个数都至少有一个是`0`，所以按位与的结果一定是`0`。
* 对于第`k+1`位，由于结果中这一位置是`1`，而 `(1<<k)` 的这一位置也是1，所以按位与的结果是1。
  因此，最终按位与的结果就是 `(1<<k)` 这个加数本身。这是因为除了 `(1<<k)`
  自身所在的位之外，其它所有位都被清零了，保留下来的只有 `(1<<k)` 这一位。
  然后，我们不难发现，将**某一个加数**拓展到**某几个加数之和**，按位与的结果就是**这几个加数之和**。

## 解决方案

根据以上的结论，我们的解决方案已经呼之欲出了。我们来看一下以下的案例。
我们用`vue3`进行示例，假设现在我们有如下规格列表：

```javascript
const specList = [
    {title: "颜色", list: ["红色", "紫色"]},
    {title: "套餐", list: ["套餐一", "套餐二"]},
    {title: "内存", list: ["64G", "128G", "256G"]},
    {title: "尺寸", list: ["5.5寸", "6寸", "6.5寸"]}
]
```

可选的规格组合为：

```javascript
const specCombinationList = ref([
    {id: '1', specs: ['紫色', '套餐一', '64G', '5.5寸']},
    {id: '2', specs: ['紫色', '套餐一', '128G', '5.5寸']},
    {id: '3', specs: ['紫色', '套餐二', '128G', '6寸']},
    {id: '4', specs: ['红色', '套餐一', '128G', '6寸']},
    {id: '4', specs: ['红色', '套餐二', '256G', '6.5寸']}
])
```

我们可以先将规格列表中的所有属性放到`Map`中，键为属性名，值为按照顺序生成的位运算的值，暂且叫做`权重`, 即:

```javascript
const specMap = ref({});
specMap.value = {
    '红色': 1, // 1 << 0
    '紫色': 2, // 1 << 1
    '套餐一': 4, // 1 << 2
    '套餐二': 8, // 以此类推...
    '64G': 16,
    '128G': 32,
    '256G': 64,
    '5.5寸': 128,
    '6寸': 256,
    '6.5寸': 512 // 1 << 9
}
```

接下来我们将所有可选列表中的规格组合进行位运算后相加的结果放到其`weight`属性中，即：

```javascript
specCombinationList.value = [
    {id: '1', specs: ['紫色', '套餐一', '64G', '5.5寸'], weight: 150},
    {id: '2', specs: ['紫色', '套餐一', '128G', '5.5寸'], weight: 166},
    {id: '3', specs: ['紫色', '套餐二', '128G', '6寸'], weight: 298},
    {id: '4', specs: ['红色', '套餐一', '128G', '6寸'], weight: 293},
    {id: '5', specs: ['红色', '套餐二', '256G', '6.5寸'], weight: 585}
]
```

然后我们可以定义一个变量，用来保存当前已选中的属性和权重。

```javascript
const selectedSpecs = reactive([]);
```

接下来考虑以下步骤：

1. 初始状态，所有规格属性都没有被选中，此时`selectedSpecs`为空数组。此时所有属性均为可选状态。
2. 当选中了`红色`，其权重为`1`，`selectedSpecs`变为`[{name:'红色', weight:1}]`。这时，对于其他属性，我们需要确定其是否能够被选中。
3. 例如，对于`紫色`，其权重为`2`，假如我们选择了`紫色`，那么当前选中的权重为`1+2=3`,这时我们通过遍历`specCombinationList`，
   将该数组中的`weight`属性与`3`进行按位与运算，如果结果是`3`
   ，说明是可被选中的，否则不能被选中。以此类推，对于`套餐一`,`套餐二`,`64G`,
   `128G`,`256G`,`5.5寸`,`6寸`,`6.5寸`这些属性都进行遍历，如果其权重与数组中的`weight`属性进行按位与运算，结果是其权重本身，那么该属性就是可选的。
4. 通过上述遍历后，我们发现`紫色`,`64G`和`5.5寸`是不可选的。这与`specCombinationList`相符。
5. 然后我们开始从以下可选的属性中去选一个属性，比如`套餐一`，此时`selectedSpecs`
   变为`[{name:'红色', weight:1}, {name:'套餐一', weight:4}]`。
   这时我们按照`步骤3`进行遍历，会发现当选择`红色`，`套餐一`后，只有`128G`和`6寸`可选了。这正好对照着`specCombinationList`
   中`id`为4的元素。
   代码实现如下：
    ```javascript
    const disabled = computed(() => {
        // 每次选中之后要判断其他规格是否能够被选中
        return (item) => {
            const weights = selectedSpecs.reduce((prev, current) => prev + current.weight, 0);
            // 所有是初始状态，则所有属性均可选，因此disabled为false.
            if (weights === 0) return false;
            const current = specMap.value[item];
            const weight = current.weight;
            // 假如选择该属性，这是选中后的“权重之和”
            let sum = weight + weights;
            for (let i = 0; i < specCombinationList.value.length; i++) {
                const sWeight = specCombinationList.value[i].weight;
                // 与“权重之和”按位与运算后与其进行比较，如果相等，说明该属性是可选的
                if ((sWeight & sum) === sum) {
                    return false;
                }
            }
            // 否则，如果判断是否是已经被选中了，如果已经被选中了，可以取消选中。
            return selectedSpecs.findIndex(o => o.name === item) == -1;
        }
    })
    ```
6. 接下来我们来实现选择逻辑：
    ```javascript
    const select = (item: string) => {
        const spec = specMap.value[item];
        const index = selectedSpecs.findIndex(o => o.name === item);
        if (index > -1) {
            selectedSpecs.splice(index, 1);
            return;
        }
        selectedSpecs.push({ name: item, weight: spec.weight });
    }
    ```

这时我们可以实现效果了，如图所示：
![SKU未考虑同一维度效果](https://www.samoy.site/sku/sku-1.gif)

## 优化
上述示例中，我们发现当选择`红色`后，`紫色`便不能再选择，这对于实际应用场景是不合理。即同一纬度情况下，应该是能够被选择的（需符合可选规格列表的某几项）。
因此我们需要对代码进行优化。还记得我们之前的`specMap`吗？我们需要对其进行改造，将其改造为以下结构：
```typescript
type SpecMap = Record<string, { weight: number, sort: number }>
```
添加了一个`sort`属性，用来表示该属性的维度。例如`红色`和`紫色`属于同一纬度，因此它们的`sort`为`0`，`套餐一`和`套餐二`属于同一维度，它们的`sort`为`1`。
这次我们的`specMap`将变为：
```javascript
const specMap = ref({
    "红色": {
        "sort": 0,
        "weight": 1
    },
    "紫色": {
        "sort": 0,
        "weight": 2
    },
    "套餐一": {
        "sort": 1,
        "weight": 4
    },
    "套餐二": {
        "sort": 1,
        "weight": 8
    },
    "64G": {
        "sort": 2,
        "weight": 16
    },
    "128G": {
        "sort": 2,
        "weight": 32
    },
    "256G": {
        "sort": 2,
        "weight": 64
    },
    "5.5寸": {
        "sort": 3,
        "weight": 128
    },
    "6寸": {
        "sort": 3,
        "weight": 256
    },
    "6.5寸": {
        "sort": 3,
        "weight": 512
    }
})
```
接下来我们对`disabled`函数进行改造：
```javascript
const disabled = computed(() => {
    return (item) => {
        // 每次选中之后要判断其他规格是否能够被选中
        const weights = selectedSpecs.reduce((prev, current) => prev + current.weight, 0);
        // 所有是初始状态，则所有属性均可选，因此disabled为false.
        if (weights === 0) return false;
        const current = specMap.value[item];
        const weight = current.weight;
        // 假如选择该属性，这是选中后的“权重之和”
        let sum = weight + weights;
        // 找到是否存在同一维度的规格已被选中的规格
        const existEle = selectedSpecs.find(o => o.sort === current.sort);
        if (existEle) {
            // 如果存在，则减去即前选中的规格权重，即替换为当前权重。
            sum -= existEle.weight;
        }
        for (let i = 0; i < specCombinationList.value.length; i++) {
            const sWeight = specCombinationList.value[i].weight;
            // 与“权重之和”按位与运算后与其进行比较，如果相等，说明该属性是可选的
            if ((sWeight & sum) === sum) {
                return false;
            }
        }
        return selectedSpecs.findIndex(o => o.name === item) == -1;
    };
})
```
然后对`select`函数进行改造：
```javascript
const select = (item) => {
  const spec = specMap.value[item];
  const eleIndex = selectedSpecs.findIndex(o => o.name === item);
  const sortIndex = selectedSpecs.findIndex(o => o.sort === spec.sort);
  if (eleIndex > -1) {
    selectedSpecs.splice(eleIndex, 1);
    return;
  }
  if (sortIndex > -1) {
    selectedSpecs.splice(sortIndex, 1);
  }
  selectedSpecs.push({ name: item, weight: spec.weight, sort: spec.sort });
}
```
这下我们大功告成了。
