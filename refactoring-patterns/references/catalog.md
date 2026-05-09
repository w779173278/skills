# 核心重构手法目录

精选《重构》第 2 版第 6–12 章中**最高频**的 30 个手法。每条按四段式：**触发 / 做法 / 范例 / 陷阱**。其余手法（约 30+ 个）请回查原书；本目录覆盖 80% 日常场景。

> 通用约定：每一步之后都跑一次测试。测试红了立即 revert。所有示例为 JavaScript。

## 目录

**第一组（基础）**：[提炼函数](#提炼函数) · [内联函数](#内联函数) · [提炼变量](#提炼变量) · [内联变量](#内联变量) · [改变函数声明](#改变函数声明) · [封装变量](#封装变量) · [变量改名](#变量改名) · [引入参数对象](#引入参数对象) · [函数组合成类](#函数组合成类) · [拆分阶段](#拆分阶段)

**封装**：[封装记录](#封装记录) · [封装集合](#封装集合) · [以对象取代基本类型](#以对象取代基本类型) · [以查询取代临时变量](#以查询取代临时变量) · [提炼类](#提炼类) · [隐藏委托关系](#隐藏委托关系) · [移除中间人](#移除中间人)

**搬移**：[搬移函数](#搬移函数) · [搬移字段](#搬移字段) · [拆分循环](#拆分循环) · [以管道取代循环](#以管道取代循环) · [移除死代码](#移除死代码)

**条件逻辑**：[分解条件表达式](#分解条件表达式) · [合并条件表达式](#合并条件表达式) · [以卫语句取代嵌套条件](#以卫语句取代嵌套条件) · [以多态取代条件表达式](#以多态取代条件表达式) · [引入特例](#引入特例) · [引入断言](#引入断言)

**API**：[将查询函数和修改函数分离](#将查询函数和修改函数分离) · [移除标记参数](#移除标记参数) · [保持对象完整](#保持对象完整) · [以查询取代参数](#以查询取代参数)

**继承**：[函数上移](#函数上移) · [以子类取代类型码](#以子类取代类型码) · [折叠继承体系](#折叠继承体系)

---

## 提炼函数

### 触发
- 一段代码需要解读才能理解"做什么"
- 函数中有注释解释一段代码块
- 含重复片段

### 做法
1. 创建新函数，按"做什么"命名（不是"怎么做"）
2. 把代码从源函数复制到新函数
3. 检查是否引用了源函数作用域内的变量；若是，作为参数传入
4. 编译（如适用）
5. 在源函数中用调用替换原代码
6. 测试
7. 检查别处是否有相似代码 → 也调用新函数

### 范例
```js
// Before
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}

// After
function printOwing(invoice) {
  printBanner();
  const outstanding = calculateOutstanding();
  printDetails(invoice, outstanding);
}
function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
```

### 陷阱
- 提炼后名字不比原代码更好 → 不该提炼，回退
- 局部变量被多次赋值且要"传出去" → 先用 拆分变量 / 以查询取代临时变量 简化
- 提炼出的函数有副作用 → 名字必须暗示副作用，且考虑 将查询函数和修改函数分离

---

## 内联函数

提炼函数的反向。

### 触发
- 函数体已和函数名同样清楚
- 一组组织糟糕的小函数，先全部内联回去再重新提炼
- 间接性没带来回报（冗赘的元素）

### 做法
1. 检查函数没被多态覆写
2. 找出所有调用点
3. 把每个调用替换为函数体
4. 测试每一处
5. 全部替换完后删掉函数定义

### 陷阱
- 多态/继承中的方法不能简单内联（子类覆写会丢）
- 调用点很多时，一次替换一个并测试，不要批量

---

## 提炼变量

### 触发
- 一个表达式难懂或需重复使用

### 做法
1. 取值表达式到不可变临时变量
2. 用变量替换原表达式（一处一处地替）
3. 测试

### 范例
```js
// Before
return order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100);

// After
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

### 陷阱
- 变量名不比表达式更好 → 不要提炼
- 表达式有副作用 → 提炼会改变求值时机；要小心

---

## 内联变量

### 触发
- 变量名没比表达式提供更多信息

### 做法
将所有引用替换回表达式；删除变量声明；测试。

---

## 改变函数声明

最常用的"改变接口"重构。涵盖：函数改名 / 增减参数 / 改参数顺序。

### 做法（简单做法）
1. 修改函数声明
2. 修改所有调用点
3. 测试

### 做法（迁移式做法，调用方多/不可一次改完时）
1. 必要时先用 提炼函数 隔离要改的部分
2. **新建** 一个签名理想的新函数
3. 让新函数调用旧函数（或反之）；保留两个并存
4. 一处一处地把调用切到新函数；每切一处测试一次
5. 全部切完，删掉旧函数

### 陷阱
- 公开 API 直接改会破坏调用方 → 用迁移式做法 + deprecation 标记
- 改名时只改了一部分调用点 → IDE 重构 / 全局 grep 兜底

---

## 封装变量

### 触发
- 全局/模块级数据被各处直接读写
- 一切结构调整的预备步骤："先封装再说"

### 做法
1. 写 getter/setter（即使数据是字段也包一层）
2. 找出所有读 → 改为 getter；找出所有写 → 改为 setter
3. 限制原数据的可见性（私有化）
4. 测试

### 范例
```js
// Before
let defaultOwner = { firstName: "Martin", lastName: "Fowler" };

// After
let defaultOwnerData = { firstName: "Martin", lastName: "Fowler" };
export function defaultOwner()       { return defaultOwnerData; }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }
```

---

## 变量改名

### 做法
1. 若变量被广泛使用 → 先 封装变量
2. 修改名字，更新所有引用；测试
3. （可选）解封装

---

## 引入参数对象

### 触发
- 一组数据成群结队出现在多处参数列表（数据泥团）

### 做法
1. 创建新结构（类或对象字面量）
2. 测试
3. 改变函数声明 在末尾加上"接收新结构"的参数（先共存）
4. 一处一处把调用方改为传入新结构；测试
5. 删掉旧的"散装"参数
6. 把使用旧字段的地方改为使用新结构的字段

### 范例
```js
// Before
function amountInvoiced(startDate, endDate) { ... }
function amountReceived(startDate, endDate) { ... }
function amountOverdue(startDate, endDate)  { ... }

// After
class DateRange { constructor(start, end) { this._start = start; this._end = end; } }
function amountInvoiced(aDateRange) { ... }
```

### 陷阱
- 新结构只是"一袋子字段"则收益有限 → 给它行为（值对象的好处才显现）

---

## 函数组合成类

### 触发
- 多个函数共享同一组参数 → 这些参数应该是一个对象

### 做法
1. 用 封装记录 把共享数据捆成对象
2. 把每个函数 搬移函数 进来，作为方法
3. 各 getter/setter 也搬进来；按需要 内联函数 简化

---

## 拆分阶段

### 触发
- 一段代码做了两类不同的事，可以分先后（如：先解析、再处理）

### 做法
1. 在两阶段间画一条线
2. 提炼一个"中转数据结构"承载第一阶段输出
3. 把第一阶段提炼成函数，返回中转结构
4. 第二阶段以中转结构为输入

### 范例
```js
// Before: 解析+计算混在一起
function priceOrder(product, quantity, shippingMethod) { ... 全部混在一起 ... }

// After
function priceOrder(product, quantity, shippingMethod) {
  const priceData = calculatePricingData(product, quantity);
  return applyShipping(priceData, shippingMethod);
}
```

---

## 封装记录

### 触发
- 直接暴露的记录/对象字面量被广泛读写

### 做法
1. 用类把记录包起来（构造函数接收字面量）
2. 写 getter/setter 暴露所需字段
3. 一处一处把外部对原字面量的读写迁移到 getter/setter
4. 测试

### 陷阱
- 嵌套记录要决定每一层都包还是只包顶层
- 集合字段 → 用 封装集合

---

## 封装集合

### 触发
- 对象暴露了一个集合字段（外部可任意 push/splice）

### 做法
1. 提供 `addX` / `removeX` 方法
2. getter 返回**副本**或不可变视图（`[...arr]` / `Object.freeze`）
3. 一处一处把外部直接修改集合的代码改为调用新方法
4. 测试

---

## 以对象取代基本类型

### 触发
- 用 string/int 表达"钱""日期""电话号""范围"等领域概念

### 做法
1. 用 封装变量 把字段包起来（先有 getter/setter）
2. 创建新值类型（如 `Money` / `PhoneNumber`）
3. setter 改为构造新类型；getter 返回内部基本值或类型本身
4. 测试
5. 把 getter 改名为暗示"返回类型"的名字（`telephone()` 而非 `telephoneNumber()`）

---

## 以查询取代临时变量

### 触发
- 临时变量阻碍了 提炼函数；变量值可以独立计算

### 做法
1. 检查变量在赋值后不再被改
2. 把"算它的表达式"提成函数（查询）
3. 替换变量的引用为函数调用
4. 删除变量声明

### 陷阱
- 算它的表达式有副作用 → 不能这么做
- 算它代价高且重复使用 → 评估性能（通常不是问题）

---

## 提炼类

### 触发
- 过大的类；有内聚的字段子集（如同前缀）

### 做法
1. 决定如何分拆职责
2. 创建新类
3. 把相关字段一个个 搬移字段 过去
4. 把相关方法一个个 搬移函数 过去
5. 检查新类的接口是否合理（也许应该 移除中间人 或对称地暴露/不暴露）

---

## 隐藏委托关系

### 触发
- 客户端写出 `obj.getDept().getManager()` 之类的链

### 做法
1. 在 `obj` 上加一个委托方法 `manager()`，内部转发
2. 调用方一处一处改成 `obj.manager()`
3. 全部改完，把链中间的访问器隐藏（如可能）

---

## 移除中间人

隐藏委托关系的反向。当一个类的接口"几乎全是转发"时，把链子还给客户端，不要再藏。

---

## 搬移函数

### 触发
- 依恋情结；霰弹式修改

### 做法
1. 检查源函数依赖的所有元素，看是否需要一起搬
2. 在目标对象创建函数（先放副本）
3. 把源函数体改为转发到目标
4. 测试 → 看是否正常
5. 一处一处把调用迁移到目标
6. 调用全部迁移完，删除源函数

### 陷阱
- 函数依赖了源对象的大量状态 → 也许应该搬的不是这个函数，而是它依赖的字段（搬移字段）

---

## 搬移字段

### 触发
- 字段总是和另一个对象一起被使用；该字段所在类没有合适的方法操作它

### 做法（推荐：先封装）
1. 若字段未封装 → 封装变量
2. 在目标类添加字段和访问器
3. 修改源类的访问器，使其转发到目标类
4. 测试
5. 一处一处迁移直接读写 → 通过新位置访问
6. 删除源字段

---

## 拆分循环

### 触发
- 一个循环做了两件不同的事（统计 + 累加 + 收集都在一起）

### 做法
1. 复制整个循环
2. 在每个副本里只保留一种职责
3. 测试
4. 必要时给每个循环 提炼函数 起个名字

> 担心性能？通常不需要。如果是热点，profiler 会告诉你；那时再用 以管道取代循环 或合并回去。

---

## 以管道取代循环

### 触发
- 命令式循环可以被 filter/map/reduce 描述

### 范例
```js
// Before
const names = [];
for (const i of input) {
  if (i.job === "programmer") names.push(i.name.toUpperCase());
}

// After
const names = input
  .filter(i => i.job === "programmer")
  .map(i => i.name.toUpperCase());
```

---

## 移除死代码

未被引用的代码、永远不会执行的分支、只在测试中用到的"产品代码"。直接删。

> Git 的存在意味着"以防万一"不是理由——历史还在。

---

## 分解条件表达式

### 触发
- 长 if/else 难读；分支条件本身复杂

### 做法
对条件、then 块、else 块分别用 提炼函数 起名。

```js
// Before
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;

// After
charge = isSummer(aDate) ? summerCharge() : regularCharge();
```

---

## 合并条件表达式

### 触发
- 多个条件检查得到相同结果

### 做法
用 `&&` / `||` 合并；合并后 提炼函数 起名。

---

## 以卫语句取代嵌套条件

### 触发
- 多层 if/else 嵌套，正常路径埋在深处

### 做法
对"特殊情况"提前 return；让正常路径成为函数主干。

```js
// Before
function getPay(emp) {
  let result;
  if (emp.isSeparated)         result = { amount: 0, reasonCode: "SEP" };
  else if (emp.isRetired)      result = { amount: 0, reasonCode: "RET" };
  else                         result = computePay(emp);
  return result;
}

// After
function getPay(emp) {
  if (emp.isSeparated) return { amount: 0, reasonCode: "SEP" };
  if (emp.isRetired)   return { amount: 0, reasonCode: "RET" };
  return computePay(emp);
}
```

---

## 以多态取代条件表达式

### 触发
- 重复的 switch（同一字段在多处分支）；类型码 + 各种分支行为

### 做法
1. 为每个类型码创建一个子类
2. 把超类的工厂改为根据类型码返回对应子类
3. 把每个分支的逻辑下移到对应子类的方法重写
4. 测试

### 陷阱
- 引入继承前，先评估"组合 + 策略对象"是否更合适

---

## 引入特例

### 触发
- 多处对"空值/缺省值"做相同的特殊处理

### 做法
- 创建一个"代表特殊情况"的对象（Null Object）
- 让所有"返回 null"的地方返回这个对象
- 删除散落的 null 检查

```js
class UnknownCustomer {
  get name() { return "occupant"; }
  get plan() { return Plan.basic(); }
  get isUnknown() { return true; }
}
```

---

## 引入断言

### 触发
- 代码假设某个条件成立，但条件未显式表达

### 做法
- 用 `assert` 表达不变量
- 不要把断言当业务逻辑用——它们是给读者和调试用的

---

## 将查询函数和修改函数分离

### 触发
- 一个函数既返回值又有副作用（CQS 违例）

### 做法
1. 复制函数，命名为新的纯查询函数（去掉副作用）
2. 让原函数调用新查询拿到值；保留副作用
3. 一处一处调整调用方：先调查询拿值，再调副作用函数
4. 副作用函数不再返回值

---

## 移除标记参数

### 触发
- 一个布尔/枚举参数控制函数走哪条分支

### 做法
- 拆成多个函数，每个函数只走一条分支
- 调用方根据情况调对应函数

---

## 保持对象完整

### 触发
- 调用方从一个对象抽出多个值传给函数 → 不如直接传对象

### 做法
- 改变函数声明：传整个对象
- 函数内部按需取值

### 陷阱
- 这会扩大函数的"知识范围"，可能引入不必要耦合 → 看场景

---

## 以查询取代参数

### 触发
- 参数的值可以通过函数自己（或它能访问的对象）查询得到

### 做法
- 删除参数，函数内部改为查询
- 修改所有调用点

### 陷阱
- 引入对全局/上下文的依赖 → 反而降低可测性。要权衡

---

## 函数上移

### 触发
- 多个子类含相同方法

### 做法
1. 检查方法体在各子类中确实一致（包括依赖的字段都在父类）
2. 把方法搬到父类
3. 删除各子类的副本
4. 测试

---

## 以子类取代类型码

### 触发
- 一个字段是"类型码"，控制行为分支

### 做法
1. 用 封装变量 包住类型码字段
2. 为每种类型码创建子类
3. 用工厂函数（以工厂函数取代构造函数）按类型码返回对应子类
4. 把分支行为分别下移到子类方法
5. 测试

---

## 折叠继承体系

### 触发
- 父类与子类已无显著区别（行为/字段几乎一致）

### 做法
- 字段/方法上移或下移到一处
- 把另一边的引用改为留下的那个
- 删除多余的类
