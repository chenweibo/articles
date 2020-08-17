## react 从入门到划水 系列一

#### 快速安装

```shell
yarn create react-app blogs
```

- Create React App 是一个官方支持的创建 React 单页应用程序的方法。它提供了一个零配置的现代构建设置。
- 类型vue-cli 一个配置好了脚手架，啥都别说先装起来跑。

#### 初始目录结构

安装完当前目录会出来一个blogs的文件夹，生成以下默认目录结构。

```
blogs
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js
```



运行项目

```powershell
cd blogs  && yarn start
```

 打开 [http://localhost:3000](http://localhost:3000/) 在浏览器中查看它 项目已经跑起来了。



#### 安装react全家桶

```powershell
yarn add react-redux
yarn add react-router-dom 
```

-  市面上有 react-router-dom  和 react-router 为什么安装react-router-dom？

- react-router-dom 这个库依赖于react-router，但是它拓展了一下在浏览器环境下运行的一些功能。在使用时不需要在单独安装react-router。

  

#### 调整目录

为了完整学习react 将引入[redux](https://github.com/reduxjs/redux) 以及[react-router](http://react-guide.github.io/react-router-cn/docs/Introduction.html) 调整脚手架的目录。

```
blogs
├── README.md
├── node_modules 
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src
    ├── + component //公共组件目录
    ├── + pages     //页面目录
    ├── + utils     //工具类目录
    ├── + reducers     //Redux全局状态管理，类比为vuex
    ├── + router     // react 路由
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js //缓存资源到本地，加快项目访问速度
```





#### 创建页面

pages下创建两个基本页面

```react
//  pages/index/index.js


import React from 'react';

function Index() {
    return (
      <div className="index">
        我是首页
      </div>
    );
  }
  
  export default Index;

```



```react
//  pages/about/index.js

import React from 'react';

function About() {
    return (
      <div className="about">
        我是关于我们页面
      </div>
    );
  }
  
  export default About;
```



#### 配置路由



```react
//  router/index.js

import React from 'react';
import {BrowserRouter, Route} from 'react-router-dom';
import Index from '../pages/index';
import About from '../pages/about';


const BasicRoute = () => (
    <BrowserRouter>    
        <Route exact path="/" component={Index}/>
        <Route exact path="/about" component={About}/>      
    </BrowserRouter>
);


export default BasicRoute;
```



#### 入口文件引入路由

```react
// index.js

import React from 'react';
import ReactDOM from 'react-dom';
import './styles/index.css';
import * as serviceWorker from './serviceWorker';


import Router from './router';

ReactDOM.render(
  <React.StrictMode>
    <Router/>
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();

```



访问[http://localhost:3000](http://localhost:3000/)  以及 [http://localhost:3000/about](http://localhost:3000/)  看到我们创建的页面，即路由配置成功。



- 到这里为止页面的配置以及完成了，下面了解下react语法。



#### React JSX

它被称为 JSX，是一个 JavaScript 的语法扩展。我们建议在 React 中配合使用 JSX，JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。JSX 可能会使人联想到模版语言，但它具有 JavaScript 的全部功能。

结合了html 完整查看 [此处](https://zh-hans.reactjs.org/docs/introducing-jsx.html)





#### 

