# react.eval

## React 组件间通讯\(不使用Context 和 redux\)

### **安装**

`npm install react.eval --save`

### github:

[https://github.com/sunluna/react.eval](https://github.com/sunluna/react.eval)

#### 组件B\(被调用\)

```
import React from 'react';
import { react } from 'react.eval';

/*
如果安装了 babel-plugin-transform-decorators-legacy
https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy#readme
可以这样↓
*/
// @react.deco //// 简化写法 @react ////^1.4.7
class BComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      content : 'nothing'
    };
    /* 构造方法里初始化实例， 简化写法 react(this) ////^1.4.7
    如果使用了ES7 decorators，就不用在constructor的最后init了
    装饰器和init任选一种，如果同时使用，则会抛出错误
      */
    react.init(this);
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

#### 父容器

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
        也可以简化 react('b.changeContent',Math.random()); ////^1.4.7
        */
        react.eval('b.changeContent', Math.random());
      }}>changeOtherComponent</button>
    </p>;
  }
}
export default AComponent;
```

##### ![](/assets/t.gif)

##### 调用或初始化前需要添加引用

```
import { react } from 'react.eval';
```

##### 需要被外部调用的组件必须在 constructor 中初始化或使用ES7的decorator标注

```
react.init(this); 
// 1.4.7 版本后简化为
// react(this);
/* 如果项目中可以使用ES7的装饰器 
 babel-plugin-transform-decorators-legacy
 https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy#readme
 可以不必在constructor的最后初始化
 在 class 上方声明 @react.deco 或 @react
 */
```

##### 给需要被外部调用的组件配置id，id必须全局唯一

```
<BComponent id="b"/>
```

##### 调用已知id组件的方式.

```
react.eval('id.方法名',参数1,参数2...)
// 1.4.7 版本后简化为 react('id.方法名',参数1,参数2...)
```

##### **注意:**

**react.eval 方法的返回值为 Promise对象，如果需要获取实际返回值，需要 react.eval\(...\).then\(\(result\)=&gt;{console.dir\(result\);}\)**

**Q: 为什么react.eval 返回Promise  而不是 实际返回值**

A: 在应用了dva、redux 等框架的项目中，dispatch很容易造成大批组件重新渲染，

在渲染过程中，react.eval 要执行的方法所在的组件实例

1、有可能处于不允许setState的生命周期

2、组件实例本身可能不存在

因此 react.eval 将会延迟到组件实例存在并且可以setState的时候执行。

另外，如果组件的方法本身返回一个Promise\(嵌套\)，react.eval会一直等待最深层的Promise结束后才触发then下一步，类似ES7 await的等待效果。

**Q:如果我要立刻获得组件方法的返回值，怎么做**

A:^1.4.8 版本后

对于某些需要立即返回数据，不会修改state，并且调用时组件必定存在的场景.

let result=react.eval\(...\)**\(\)                                          //react\(...\)\(\)**

在eval方法后再加一对小括号，就可以立刻返回未经处理的方法返回值，不必使用then取得Promise的返回结果

```
 let result= react('b.changeContent','......')();
```

![](/assets/import.png)

![](/assets/imposrt.png)

如果结果确实是异步返回结果，例如各种ajax,还可以使用** await/async **改造调用的方法

![](/assets/impaasddddort.png)

![](/assets/impozzrt.png)

**Q:这个组件和redux还有Context API最大的不同是什么**

A:不遵守单一的数据流向，组件状态可根据自己业务需要分散到子组件中自己处理自己的动作，需要交互则通过react.eval，而不是统一到一个store里管理。

如果把react.eval 和 dva,redux 比作公司，那么

①dva和redux就是boss绝对掌权，员工只负责按指令展示，完全无脑，发生任何事情都必须通知boss\(dispatch\)，然后层层下发（connect,重绘）。

②react.eval 是公司分割成不同的部门，部门下还可以再继续分割子部门，部门内部的事情自己解决不用上报\(this.setState\)， 部门之间沟通，直接通知\(react.eval\)，小事不需要通知boss，不需要层层下发，变更直接发生在需要变更的部门。

# 附言

redux和dva并不是所有问题的最佳方案，尤其是在pc端的业务系统，盲目坚持单向数据流会造成大量的代码冗余，降低工作效率，bug层出，不利于维护。

dva和redux翻倍的代码量本身就是制造bug的根源。jquery，vue这些经典的东西的生存哲学依然有可取之处。

如果问题的根源就是  单向数据流 这种执念，而这种执念在某些情况下会伤害到我们的利益，那么，我们要学会变通。

组件应当 为 程序员 服务，极少的配置获取尽可能多的功能。组件存在的目的是为了通过封装组合，尽最大可能不出bug，让经验得以积累，提高效率。

react.eval 的执行方式仿照easyui\(\([http://www.jeasyui.com](http://www.jeasyui.com%29%29\)，存在的目的是为了黏合小组件，根据不同的业务需要组合出不同规格的高级组件。

作者是程序员中的搬砖工。

盖房子的时候不是每次都需要一砖一瓦地堆砌，我们应当通过总结经验，将砖瓦预先制作成更高级的东西。

第一次盖房子可能确实是一砖一瓦慢慢盖起来的。

第二次、第三次 我们不妨开吊车把定制尺寸的地基、墙面、屋顶直接带过来组合到一起，以最快、最安全、最高效的方式把房子盖好。

