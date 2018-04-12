# react.eval

## React 组件间通信

### 不借助Context 和 redux

**调用组件内部方法，定点精准更新组件状态**

**制作易于重用并且自由交互的react组件**

### **安装**

`npm install react.eval --save`

### github:

[https://github.com/sunluna/react.eval](https://github.com/sunluna/react.eval)

### 示例（同级组件交互）

#### 组件B\(被调用\)

```
import React from 'react';
import { react } from 'react.eval';

/*
如果安装了 babel-plugin-transform-decorators-legacy
https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy#readme
可以这样↓
*/
//@react 
class BComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      content : 'nothing'
    };
    /* 构造方法初始化实例
    如果使用了ES7 decorators，就不用在constructor初始化
    装饰器和constructor初始化任选一种，如果同时使用，则会抛出错误
      */
    react(this);
  }
  // 提供给外界调用的方法
  changeContent = (str) => {
    console.log("changing"+str);
    this.setState({
      content:str
    });
  }
  render() {
    return <p>
      Component {this.id}:
      {this.state.content}
    </p>;
  }
}
export default BComponent;
```

#### 容器

```
import React from 'react';
import AComponent from './eva';
import BComponent from './evb';

class TEval extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    // 必须给被调用的组件设置一个全局唯一的id才能进行调用
    return (
      <div>
        <AComponent />
        <BComponent id="b"/>
      </div>
    );
  }
}
export default TEval;
```

#### 组件A\(调用者\)

```
import React from 'react';
import { react } from 'react.eval';

class AComponent extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <p>
      <button onClick={() => {
        /* 在事件方法中 使用 id.方法名 参数1、参数2...方式调用B组件提供的方法
        */
        react('b.changeContent', Math.random());
      }}>changeOtherComponent</button>
    </p>;
  }
}
export default AComponent;
```

##### ![](/assets/t.gif)

### 步骤

##### 调用或初始化前需要添加引用

```
import { react } from 'react.eval';
```

##### 需要被外部调用的组件必须在 constructor 中初始化或使用ES7的decorator标注，组件必须是有状态的

```
react(this);
/* 如果项目中可以使用ES7的装饰器 
 babel-plugin-transform-decorators-legacy
 https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy#readme
 则不必在constructor中初始化
 在 class 上方声明  @react
 */
```

##### 给需要被外部调用的组件配置id，id必须全局唯一

```
<BComponent id="b"/>
```

##### 调用已知id组件的方式.

```
react.eval('id.方法名',参数1,参数2...)
// 可以简化为 react('id.方法名',参数1,参数2...)
```

##### **注意:**

**react.eval 方法的返回值为 Promise对象，如果需要获取实际返回值，需要 react.eval\(...\).then\(\(result\)=&gt;{console.dir\(result\);}\)**

**Q: 为什么react.eval 返回Promise  而不是 实际返回值**

A: 在应用了dva、redux 等框架的项目中，dispatch会引发组件重新渲染，

在渲染过程中，react.eval 要执行的方法所在的组件实例

1、有可能处于不允许setState的生命周期

2、组件实例本身可能不存在

因此 react.eval 将会延迟到组件实例存在并且可以setState的时候执行。

另外，如果组件的方法本身返回一个Promise\(嵌套\)，react.eval会一直等待最深层的Promise结束后才触发then下一步，类似ES7 await的等待效果。

**Q:如果我要立刻获得组件方法的返回值，怎么做**

A:对于某些需要立即返回数据，不会修改state，并且调用时组件实例必定存在的场景.

let result=react.eval\(...\)**\(\)                                          //react\(...\)\(\)**

在eval方法后再加一对小括号，就可以立刻返回未经处理的方法返回值，不必使用then取得Promise的返回结果

```
 let result= react('b.changeContent','......')();
```

如果组件实例调用时一定存在，还可以直接获取组件实例，然后用组件实例调用指定方法

```
let result= react('b').changeContent('......');
```

![](/assets/import.png)![](/assets/20180408133437.png)

![](/assets/imposrt.png)

![](/assets/20180408133126.png)

如果结果确实是异步返回结果，例如各种ajax,还可以使用** await/async **改造调用的方法

![](/assets/impaasddddort.png)

![](/assets/impozzrt.png)

**Q:react.eval 和 redux/dva这些兼容性怎么样?**

A:可以直接与redux/dva混用，没有冲突。

甚至为了兼容项目使用redux/dva，还对react.eval方法进行了调整适应\(见:**为什么react.eval 返回Promise  而不是 实际返回值**\)。

可以在 React\(15.x,16.x\)上无缝使用。

**Q:react.eval的优势有哪些?**

A: react.eval 实现组件传值调用，借鉴传统的jQuery组件设计风格，代码量少，并且代码逻辑集中在被封装的组件单个js中，便于维护。同一界面多处复用组件时，只需要指定不同的 id 即可准确调用指定组件内部的方法。组件之间不受组件层级或位置制约，任意访问。可以以搭积木的方式封装复合组件。API少，方法名好记。

**Q:react.eval的适用范围**

A:react.eval 可以做到像dva、redux一样的跨组件通讯，只不过实现原理不同。

如果项目不是整体dva/redux的，react.eval可以承担所有的组件通信需求。

如果项目是整体dva/redux的，建议最多只在封装组件和复合组件的时候使用\(或者不用\)，只做纯技术封装，可以提高组件复用能力。

不建议在dva/redux的store和reducers里面出现，以免造成混乱。

# 附言

组件应当为 程序员 服务，用极少的配置获取尽可能多的功能。

组件存在的意义是通过封装组合，尽最大可能不出bug，让经验得以积累，提高效率，降低成本。

**react.eval 存在的价值是，允许定点精准更新目标组件，从而制作易于重用并且可以自由交互的react组件**

# [版本变更](/ban-ben-bian-geng.md)



