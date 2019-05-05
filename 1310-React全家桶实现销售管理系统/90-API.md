---
show: step
version: 1.0
enable_checker: true
---

# API 

## 一、实验介绍

#### 1.1 实验内容

在上一节实验中，我们完成了数据库的配置和数据的添加，但是现在我们还没有办法获取到我们添加的那些数据，也没有办法向数据库提交我们的表单数据，在本节实验过后，这些功能就都可以实现了。

#### 1.2 实验知识点

+ express
+ axios

#### 1.3 实验环境

+ `Theia`:  一款前后端分离的、基于 web 的 云 IDE。
+ `Mini Browser`：浏览器，右键点击目标 html 文件，选择 open with 下的 Mini Browser 即可在 IDE 中打开浏览器。

#### 1.4 适合人群

本课程难度一般，适合具有 React 基础的用户，熟悉 React 的基础知识，通过学习 React 的全家桶的使用以及 express 框架和 mongoDB 的简单使用，完成一个完整的 React 项目。

## 二、实验步骤

接口的使用过程我们可以这样来理解，前端发出请求，后端接收到请求，后端请求数据库，数据库返回信息，后端将信息返回给前端。由此我们针对每一个过程，可以将它们分模块处理，我们先来完成前端发出请求的部分。

### 2.1 前端部分

前后端交互会涉及到 http 请求，本项目中我们通过封装一些 axios 请求，来提供 http 请求的方法。

在 `react-pxq/src` 下创建文件夹 `api`:

```bash
$ mkdir api
$ cd api
```

然后我们在这个文件夹里创建我们的封装 axios 的文件和调用这些封装的方法进行请求的 api 文件：

```bash
$ touch ajax.js api.js
```

`ajax.js`

```js
import axios from 'axios'

export function getAjax(url,cb) {
    axios.get(url)
    .then(function (response) {
        cb(response);
    })
    .catch(function (error) {
        console.log(error);
    });
}

export function putAjax(url,data,cb) {
    axios.put(url,data)
    .then(function (response) {
        cb(response);
    })
    .catch(function (error) {
        console.log(error);
    });
}

export function postAjax(url,data,cb) {
    axios.post(url,data)
    .then(function (response) {
        cb(response);
    })
    .catch(function (error) {
        console.log(error);
    });
}

export function deleteAjax(url,cb) {
    axios.delete(url)
    .then(function (response) {
        cb(response);
    })
    .catch(function (error) {
        console.log(error);
    });
}
```

我们封装了四种 http 使用 axios 的请求，它们的形式大致相同，我们以 get 请求为例，做一个讲解：

```js
export function getAjax(url,cb) {
    axios.get(url)
    .then(function (response) {
        cb(response);
    })
    .catch(function (error) {
        console.log(error);
    });
}
```

封装后的方法有两个参数，分别是 url 和 cb。url 比较容易理解，这是我们将要请求的 url。然后我们调用 axios.get(url),发起一个 get 请求，接着我们使用 then() 异步的调用一个回调函数 ：

```js
function (response) {
    cb(response);
}
```

这个回调函数中使用了另外一个参数，cb。cb 是一个方法，用于处理我们的返回结果，对 redux 理解比较好的同学可能已经猜出来了，这里我们要传入的就是触发 action 的 dispatch 方法，我们在调用接口请求到一些数据后，总要把这些数据传给 store，由 store 统一管理。

下面，我们就来编写接口函数：

`/react-pxq/src/api/api.js`

```js
import * as Ajax from '@/api/ajax';
import {getProData} from '@/store/production/action';
import {getRecord} from '@/store/record/action';

//获取对应类型的 record
export const getrecordDetail = (type) => {
    return function(dispatch){
    Ajax.getAjax(`/api/record?type=${type}`,function(response){
      if(response){
        dispatch(getRecord(response.data));
      }
    })
  }
}

//提交表单到 record
export const uploadData = (data) => {
    Ajax.postAjax(`/api/record`,data,function(response){
        if(response){
            return response.data;
        }
    })
}

//获取 production 信息
export const getproDetail = () => {
  return function(dispatch){
    Ajax.getAjax(`/api/production`,function(response){
      if(response){
        response.data.map(item => {
          item.selectStatus = false;
          item.selectNum = 0;
          return item;
        })
        dispatch(getProData(response.data));
      }
    })
  }
}
```

在接口文件中，我们首先将封装好的 axios import as Ajax 进来，方便接下来的调用，然后导入两个 action，getProData 和 getRecord，这两个 action 对应的 reducer 都是将获取到的新的数据返回，传给 store。下面我们来看接口具体的代码：

```js
export const getrecordDetail = (type) => {
    return function(dispatch){
    Ajax.getAjax(`/api/record?type=${type}`,function(response){
      if(response){
        dispatch(getRecord(response.data));
      }
    })
  }
}
```

还记得我们 record 有三种类型的记录吗，我们在请求获取 record 数据的时候，并不请求所有的数据，而是请求对应 type 类型的数据，所以这个接口有一个参数 type。然后这个接口发起了一个 get 请求 `Ajax.getAjax(url,cb)`,这里的 cb 对应的就是：

```js
function(response){
    if(response){
        dispatch(getRecord(response.data));
    }
}
```

就像我们之前讲解的那样，这个回调函数用 get 请求返回的数据 response 作为参数，用 dispatch 触发 action。另外两个接口的原理比较类似，我们不作赘述。

### 2.2 后端部分

接口的前端部分完成了，现在我们来编写后端部分的代码。

#### 2.2.1 逻辑处理

我们先来编写后端接收到请求后的处理逻辑，在 `react-pxq/server` 下创建文件夹 `models`

```bash
$ mkdir models
$ cd models
```

因为我们的接口并不多，所以将所有的接口逻辑处理放在一个文件是可以的，在 `models` 下创建文件 `home.js`

```bash
$ vim home.js
```

`home.js`

```js
let Production = require('../schemas/production');
let Record = require('../schemas/record');

exports.insertRecord = (req,res,next) => {
    Record.create(req.body,function(err,result){
        res.send(result);
    })
}

exports.getRecord = (req,res,next) => {
    Record.find({'type':req.query.type},function(err,result){
        console.log(result);
        res.send(result);
    })
}

exports.getProduction = (req,res,next) => {
    Production.find({},function(err,result){
        res.send(result);
    })
}
```

与前端的逻辑相对应，我们接口的逻辑处理在后端也有三块，这里的逻辑处理主要是数据库的部分，因为我们的数据处理已经在前端使用 redux 完成了，后端负责的只有从数据库查询数据或者是插入数据。

首先，因为涉及到数据库的操作，所以我们把之前建立的两个数据模型导入到文件中，

+ insertRecord

```js
exports.insertRecord = (req,res,next) => {
    Record.create(req.body,function(err,result){
        res.send(result);
    })
}
```

上面的逻辑是数据库的插入操作，用于将提交的表单数据插入到数据库中，它对应的前端接口代码如下：

```js
export const uploadData = (data) => {
    Ajax.postAjax(`/api/record`,data,function(response){
        if(response){
            return response.data;
        }
    })
}
```

前端发起了一个 post 请求，参数为 data，在后端部分，我们可以通过 req.body 获取到这个 data 参数，然后调用 create() 方法，向表 Record 中插入 data 数据，后面的回调 function 用于返回处理结果信息。

+ getRecord

```js
exports.getRecord = (req,res,next) => {
    Record.find({'type':req.query.type},function(err,result){
        console.log(result);
        res.send(result);
    })
}
```

上面的逻辑代码是数据库的查询操作，查询字段 type = req.query.type 的数据返回，它对应前端接口代码如下：

```js
export const getrecordDetail = (type) => {
    return function(dispatch){
    Ajax.getAjax(`/api/record?type=${type}`,function(response){
      if(response){
        dispatch(getRecord(response.data));
      }
    })
  }
}
```

就是我们前端接口中作了讲解的部分，这是一个 get 请求，所以参数 type 要在 req.query.type 中获取。

+ getProduction

```js
exports.getProduction = (req,res,next) => {
    Production.find({},function(err,result){
        res.send(result);
    })
}
```

上面的逻辑代码是数据库的查询操作，查询表 Production 中的所有数据进行返回。

以上就是我们后端的逻辑处理部分。

#### 2.2.2 路由处理

当前端发出请求，服务端需要解析前端请求的 url，然后根据 url 调用不同的处理逻辑，这里我们使用 express 为我们提供的 Router() 来完成。

在 `routes` 下创建我们的路由处理文件 `user.js`,并删除其它的路由文件

```bash
$ rm users.js index.js
$ vim user.js
```

`user.js`

```js
let express = require('express');

let router = express.Router();

let home = require('../models/home')

router.post(`/record`,home.insertRecord);

router.get(`/record`,home.getRecord);

router.get(`/production`,home.getProduction);

module.exports = router;

```

路由 user.js 的逻辑很简单，分别引入 express 和 我们之前编写的逻辑处理文件 home。对不同类型的请求以及不同的 url，我们调用不同的逻辑处理。

然后我们修改 `/react-pxq/server/app.js` 文件如下：

```js
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');

var mongoose = require('mongoose'),
    DB_URL = 'mongodb://localhost:27017/myapp';

mongoose.Promise = global.Promise;

//连接
mongoose.connect(DB_URL, {useMongoClient: true,});

//连接成功
mongoose.connection.on('connected', function () {
    console.log('connect success ' + DB_URL);
});

//连接异常
mongoose.connection.on('error',function (err) {
    console.log('connect error: ' + err);
});

//连接断开
mongoose.connection.on('disconnected', function () {
    console.log('disconnect');
});

var app = express();

if (process.env.NODE_ENV !== 'production') {
    var webpack = require('webpack');
    var webpackConfig = require('../config/webpack.config.js');
    var webpackCompiled = webpack(webpackConfig);
    // 配置运行时打包
    var webpackDevMiddleware = require('webpack-dev-middleware');
    //console.log(webpackConfig.output.publicPath);
    app.use(webpackDevMiddleware(webpackCompiled, {
        publicPath: webpackConfig.output.publicPath,
        stats: {colors: true},
        lazy: false,
        watchOptions: {
            poll: true
        },
    }));

    // 配置热更新
    var webpackHotMiddleware = require('webpack-hot-middleware');
    app.use(webpackHotMiddleware(webpackCompiled));
}


// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.all('*',function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
    res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By",' 3.2.1')
    res.header("Content-Type", "application/json;charset=utf-8");
    next();
});

let user = require('./routes/user')
app.use('/api', user);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;

```

### 2.3 组件调整

在之前的实验中，我们为了展示效果，在前端组件中没有调用 API，数据的展示大多是在组件中定义，然后渲染出来，现在我们需要修改对应的页面，在页面中加入 API 的调用。

#### 2.3.1 `/react-pxq/src/pages/home/home.jsx`

```js
import React, { Component } from 'react';
import { Link } from 'react-router-dom';
import { connect } from 'react-redux';
import { is, fromJS } from 'immutable';
import PropTypes from 'prop-types';
import { uploadData } from '@/api/api';
import { saveFormData, clearData } from '@/store/home/action';
import { clearSelected } from '@/store/production/action';
import PublicHeader from '@/components/header/header';
import PublicAlert from '@/components/alert/alert';
import TouchableOpacity from '@/components/TouchableOpacity/TouchableOpacity';
import mixin, { padStr } from '@/utils/mixin';
import './home.less';


@mixin({padStr})
class Home extends Component {
  static propTypes = {
    formData: PropTypes.object.isRequired,
    saveFormData: PropTypes.func.isRequired,
    clearData: PropTypes.func.isRequired,
    clearSelected: PropTypes.func.isRequired,
  }

  state = {
    alertStatus: false, //弹框状态
    alertTip: '', //弹框提示文字
  }

  selectedProList = [];

  handleInput = (type, event) => {
    let value = event.target.value;
    switch(type){
      case 'orderSum':
        value = value.replace(/\D/g, '');
      break;
      case 'name':
      break;
      case 'phoneNo':
        value = this.padStr(value.replace(/\D/g, ''), [3, 7], ' ', event.target);
      break;
      default:;
    }
    this.props.saveFormData(value,type);
  }


  sumitForm = () => {
    const {orderSum, name, phoneNo} = this.props.formData;
    //填充要提交的数据
    let recordData = {};
    recordData.customers_name = this.props.formData.name;
    recordData.phoneNo = this.props.formData.phoneNo;
    recordData.created_at = new Date().toLocaleString();
    recordData.product = this.selectedProList;
    //计算金额和佣金
    recordData.sales_money = 0;
    recordData.commission = 0;
    recordData.product.map((item)=>{recordData.sales_money+=item.product_price*item.selectNum;return recordData.commission+=item.commission*item.selectNum});
    recordData.type = "audited";
    recordData.tip = "等待管理员审核，审核通过后，佣金将结算至账户";
    let alertTip = '';
    if(!orderSum.toString().length){
      alertTip = '请填写金额';
    }else if(!name.toString().length){
      alertTip = '请填写姓名';
    }else if(!phoneNo.toString().length){
      alertTip = '请填写正确的手机号';
    }else{
      uploadData(recordData);
      alertTip = '添加数据成功';
      this.props.clearSelected();
      this.props.clearData();
    }
    this.setState({
      alertStatus: true,
      alertTip,
    })
  }

  // 关闭弹款
  closeAlert = () => {
    this.setState({
      alertStatus: false,
      alertTip: '',
    })
  }

  // 初始化数据，获取已选择的商品
  initData = props => {
    this.selectedProList = [];
    props.proData.dataList.forEach(item => {
      if(item.selectStatus && item.selectNum){
        this.selectedProList.push(item);
      }
    })
  }

  componentWillReceiveProps(nextProps){
    if(!is(fromJS(this.props.proData), fromJS(nextProps.proData))){
      this.initData(nextProps);
    }
  }

  shouldComponentUpdate(nextProps, nextState) {
    return !is(fromJS(this.props), fromJS(nextProps)) || !is(fromJS(this.state),fromJS(nextState))
  }

  componentWillMount(){
    this.initData(this.props);
  }


  render() {
    return (
      <main className="home-container">
        <PublicHeader title='首页' record />
        <p className="common-title">请录入您的信息</p>
        <form className="home-form">
          <div className="home-form-tiem">
            <span>销售金额：</span>
            <input type="text" placeholder="请输入订单金额" value={this.props.formData.orderSum} onChange={this.handleInput.bind(this, 'orderSum')}/>
          </div>
          <div className="home-form-tiem">
            <span>客户姓名：</span>
            <input type="text" placeholder="请输入客户姓名" value={this.props.formData.name} onChange={this.handleInput.bind(this, 'name')}/>
          </div>
          <div className="home-form-tiem">
            <span>客户电话：</span>
            <input type="text" maxLength="13" placeholder="请输入客户电话" value={this.props.formData.phoneNo} onChange={this.handleInput.bind(this, 'phoneNo')}/>
          </div>
        </form>
        <div>
          <p className="common-title">请选择销售的产品</p>
          <Link to="/production" className="common-select-btn">
            {
              this.selectedProList.length ? <ul className="selected-pro-list">
                {
                  this.selectedProList.map((item, index) => {
                    return <li key={index} className="selected-pro-item ellipsis">{item.product_name}x{item.selectNum}</li>
                  })
                }
              </ul>:'选择产品'
            }
          </Link>
        </div>
        <div className="upload-img-con">
          <p className="common-title">请上传发票凭证</p>
          <div className="file-lable">
            <span className="common-select-btn">上传图片</span>
            <input type="file" />
          </div>
        </div>
        <TouchableOpacity className="submit-btn" clickCallBack={this.sumitForm} text="提交" />
        <PublicAlert closeAlert={this.closeAlert} alertTip={this.state.alertTip} alertStatus={this.state.alertStatus} />
      </main>
    );
  }
}

export default connect(state => ({
  formData: state.formData,
  proData: state.proData,
}), {
  saveFormData,
  clearData,
  clearSelected,
 }
)(Home);
```

#### 2.3.2 `/react-pxq/src/pages/production/production.jsx`

```js
import React, { Component } from 'react';
import { is, fromJS } from 'immutable';
import { connect } from 'react-redux';
import { togSelectPro, editPro } from '@/store/production/action';
import { getproDetail } from '@/api/api';
import PropTypes from 'prop-types';
import PublicHeader from '@/components/header/header';
import './production.less';

class Production extends Component{
  static propTypes = {
    proData: PropTypes.object.isRequired,
    togSelectPro: PropTypes.func.isRequired,
    editPro: PropTypes.func.isRequired,
  }

  handleEdit = (index, num) => {
    let currentNum = this.props.proData.dataList[index].selectNum + num;
    if(currentNum < 0){
      return
    }
    this.props.editPro(index, currentNum);
  }

  // 选择商品，交由redux进行数据处理，作为全局变量
  togSelect = index => {
    this.props.togSelectPro(index);
  }

  shouldComponentUpdate(nextProps, nextState) {
    return !is(fromJS(this.props), fromJS(nextProps)) || !is(fromJS(this.state), fromJS(nextState))
  }

  componentDidMount(){
    if(!this.props.proData.dataList.length){
      this.props.getproDetail();
    }
  }

  render(){
    return (
      <main className="common-con-top">
        <PublicHeader title='首页' confirm />
        <section className="pro-list-con">
          <ul className="pro-list-ul">
            {
              this.props.proData.dataList.map((item, index) => {
                return <li className="pro-item" key={index}>
                  <div className="pro-item-select" onClick={this.togSelect.bind(this, index)}>
                    <span className={`icon-xuanze1 pro-select-status ${item.selectStatus? 'pro-selected': ''}`}></span>
                    <span className="pro-name">{item.product_name}</span>
                  </div>
                  <div className="pro-item-edit">
                    <span className={`icon-jian ${item.selectNum > 0? 'edit-active':''}`} onClick={this.handleEdit.bind(this, index, -1)}></span>
                    <span className="pro-num">{item.selectNum}</span>
                    <span className={`icon-jia`} onClick={this.handleEdit.bind(this, index, 1)}></span>
                  </div>
                </li>
              })
            }
          </ul>
        </section>
      </main>
    )
  }
}


export default connect(state => ({
  proData: state.proData,
}), {
  getproDetail,
  togSelectPro,
  editPro
})(Production);
```

#### 2.3.3 `/react-pxq/src/pages/record/components/recordList.jsx`

```js
import React, { Component } from 'react';
import { is, fromJS } from 'immutable';
import {getrecordDetail} from '@/api/api';
import {connect} from 'react-redux';
import './recordList.less';
import PropTypes from 'prop-types';

class RecordList extends Component{
  static propTypes = {
    recordData:PropTypes.object.isRequired,
  }

  componentWillReceiveProps(nextProps){
    // 判断类型是否重复
    let currenType = this.props.location.pathname.split('/')[2];
    let type = nextProps.location.pathname.split('/')[2];
    if(currenType !== type){
      this.props.getrecordDetail(type);
    }
  }

  shouldComponentUpdate(nextProps, nextState){
    return !is(fromJS(this.props), fromJS(nextProps)) || !is(fromJS(this.state), fromJS(nextState))
  }

  componentWillMount(){
    let type = this.props.location.pathname.split('/')[2];
    this.props.getrecordDetail(type);
  }


  render(){
    return (
      <div>
        <ul className="record-list-con">
        {
          this.props.recordData.dataList.map((item, index) => {
            return <li className="record-item" key={index}>
              <section className="record-item-header">
                <span>创建时间：{item.created_at}</span>
                <span>{item.type_name}</span>
              </section>
              <section className="record-item-content">
                <p><span>用户名：</span>{item.customers_name} &emsp; {item.customers_phone}</p>
                <p><span>商&emsp;品：</span>{item.product.map((content)=> content.product_name)}</p>
                <p><span>金&emsp;额：</span>{item.sales_money} &emsp; 佣金：{item.commission}</p>
              </section>
              <p className="record-item-footer">{item.tip}</p>
            </li>
          })
        }
        </ul>
      </div>
    );
  }

}

export default connect(state => ({
  recordData:state.recordData,
}),{
  getrecordDetail,
})(RecordList);
```

### 2.4 优化项目

到此为止，我们的项目功能都已经实现了，我们使用 react 全家桶 + express + mongoDB 搭建了一个销售管理系统，现在我们要对它进行一些优化。

#### 2.4.1 `/react-pxq/src/index.js`

```js
import React from 'react';
import ReactDOM from 'react-dom';
import Route from './router/';
import FastClick from 'fastclick';
import {Provider} from 'react-redux';
import store from '@/store/store';
import * as serviceWorker from './serviceWorker';
import { AppContainer } from 'react-hot-loader';
import './utils/setRem';
import './style/base.css';

FastClick.attach(document.body);

const render = Component => {
  ReactDOM.render(
    <Provider store={store}>
        <AppContainer>
            <Component />
        </AppContainer>
    </Provider>,
    document.getElementById('root'),
  )
}

render(Route);

if (module.hot) {
  module.hot.accept('./router/', () => {
    render(Route);
  })
}

serviceWorker.unregister();
```

由于我们的项目是用移动端开发模式查看效果的，而移动端浏览器在派发点击事件的时候，通常会出现 300ms 左右的延迟，另外，移动端的双击会缩放导致 click 判断延迟。我们通过使用 FastClick 插件来减少这个延迟，优化我们的应用。

#### 2.4.2 `/react-pxq/src/router/index.js`

```js
import React, { Component } from 'react';
import { HashRouter, Switch, Route, Redirect } from 'react-router-dom';
import asyncComponent from '@/utils/asyncComponent';
import home from "@/pages/home/home";
const record = asyncComponent(() => import("@/pages/record/record"));
const production = asyncComponent(() => import("@/pages/production/production"));
const balance = asyncComponent(() => import("@/pages/balance/balance"));
const helpcenter = asyncComponent(() => import("@/pages/helpcenter/helpcenter"));

export default class RouteConfig extends Component{
  render(){
    return(
      <HashRouter>
        <Switch>
          <Route path="/" exact component={home} />
          <Route path="/balance" component={balance} />
          <Route path="/helpcenter" component={helpcenter} />
          <Route path="/record" component={record} />
          <Route path="/production" component={production} />
          <Redirect to="/" />
        </Switch>
      </HashRouter>
    )
  }
}
```

在路由的优化中，我们使用了 asuyncComponent 这个高级组件，它用于异步的加载组件，这样我们就可以实现按需加载组件。

## 三、总结

通过本节实验，我们完成了项目的前后端交互部分。