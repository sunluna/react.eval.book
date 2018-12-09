# 1、refs对象

import { refs } from 'react.eval';

#### refs可以获取已经注册并且调用的组件的实例

`refs.toast.show();`

请确保调用时组件实例已存在并且id匹配

&lt;Toast id='toast'/&gt;

注: refs对象并不存在于全局对象上，而是分散在各处引用的闭包内部，通过发布订阅模式维持refs对象为最新的实例id引用集合。

# 2、ref函数

import { ref } from 'react.eval';

#### ref函数提供组件方法调用、注册组件实例、方法装饰器等多种功能\(全能方法\)

| 方法 | 参数（序号，类型，说明） | 描述 |
| :--- | :--- | :--- |
| ref\('id'\) | ①  String  实例的id | 等同于 ref.get\('id'\),                                用于获取指定id的react组件实例 |
| ref\('id.methodName'\) | ① String 实例id  .  方法名                      ② ③......方法其他参数 | 等同于  ref.eval\('id.methodName'\);         用于执行实例中的方法,                            返回值为Promise |
| ref\(this\) | ① Object  组件实例 | 等同于  ref.init\(this\) ,                                用于在constructor中注册组件实例到   refs引用 |
| ref\(Toast\) | ① Function React组件（有状态组件） | 等同于 ref.deco\(Toast\),                          HOC 用于装饰普通React组件,              使组件在实例化时可以被注册到refs对象上，在销毁时可以从refs对象上移除 |
| ref.chunk\(newBase\) | ① Object \| undefined                            事件池存放的新变量位置 | 迁移公共事件池位置到某个隐藏变量,       一般不用调用此方法,                               此方法仅针对某些有代码洁癖的人。      调用后，可将公共事件池从全局移除。 |

**需要注意的地方**

**1、ref\('id.methodName',参数1,参数2,参数3....\)这种用法，经过了特殊处理，返回值为Promise**

原因:

1\) 在某些使用了dva，redux的项目上，调用方法时，组件处于不能setState的状态，或者暂时不存在，

针对这种情况，解决措施为 拦截生命周期中的钩子，如果组件不可立即使用，则等待35毫秒后重试，最多等待3秒后放弃

因此返回值为Promise

2\) 很多情况下，被调用的方法本身也可能返回一个Promise，多层Promise很是烦人，因此 增加了类似 await一样的逻辑，对结果处理，Promise的最终结果为嵌套的最深层的Promise的结果

```
getContent=()=>{
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      resolve('最终结果');
    },500);
  });
} 
...
//另一个文件
const result= await ref('toast.getContent');
// result= '最终结果'
或
ref('toast.getContent').then((result)=>{
  console.log(result);
  // 最终结果
});
```

3\) 如果你对ref\('id.methodName'\)用法的返回值处理感到纠结和厌烦，建议使用refs.id 这种方式来调用组件实例，方法结果不会经过任何处理。

也可以 ref\('id.methodName'\)\(\)  ，就是在调用方法后追加一对小括号，这样就会立刻返回组件实例方法的结果，不经过任何处理。

**2、ref.chunk\(newBase\)**

