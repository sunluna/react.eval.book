# react.eval

## React 组件间通信\(调用已知id的组件实例\)

###### 我们都知道 document.getElementById\('app'\) 可以用来获得 id 为 app 的dom元素 &lt;div id="app"&gt;&lt;/div&gt;

###### 在react环境下，本组件提供 获得指定id的 react组件实例功能  &lt;Toast id="app" /&gt;

##### 使用本组件需要安装  babel-plugin-transform-decorators-legacy

\([https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy\#readme](https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy#readme "如何安装?")\)

webpack1.x =&gt;  webpack.config.js

```
...
module:{
    loaders:[
        {
            test:/.js[x]?$/,
            exclude: /node_modules/,
            loader:'babel-loader',
            query:{
                presets:['es2015','react','stage-0'],
                plugins:['transform-runtime','transform-decorators-legacy']
            }
        }
    ]
}
```



