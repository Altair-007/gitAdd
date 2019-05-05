---
show: step
version: 1.0
enable_checker: true
---

# 使用 redux 管理 state

## 一、实验介绍

#### 1.1 实验内容

我们在之前的实验中已经介绍了 redux 的基础知识，一个使用 redux 管理 state 的 react 应用只有一个 store，所有的对 state 的操作都是通过触发 action，然后调用对应的 reducer 进行处理，返回一个新的 state 然后改变 store，组件再从 store 中获取封装了新的 state 的 props。我们在实验中使用到的 redux 更加进阶。在实验中，我们并不打算直接调用 `store.dispatch`,如果那样做我们需要在页面中先把 store import 入页面，在每次要触发 action 的时候，都要调用 `store.dispatch`,比较繁琐。但是我们又想要在 React 组件中使用 redux ，把 store 绑定到我们的视图层上，这时候就要用到我们的 React-redux 的两个模块：`Provider` 和 `connect`。下面我们来简单讲解一下这两个模块的原理和用法。

#### 1.2 实验知识点

+ React
+ Redux

#### 1.3 实验环境

+ `Theia`:  一款前后端分离的、基于 web 的 云 IDE。
+ `Mini Browser`：浏览器，右键点击目标 html 文件，选择 open with 下的 Mini Browser 即可在 IDE 中打开浏览器。

#### 1.4 适合人群

本课程难度一般，适合具有 React 基础的用户，熟悉 React 的基础知识，通过学习 React 的全家桶的使用以及 express 框架和 mongoDB 的简单使用，完成一个完整的 React 项目。

## 二、实验原理

下面我们来讲解实验原理。

### 2.1 Provider & connect

+  Provider

还记得我们说 Redux 的原理是维护单一的数据源，使用的时候将数据从上往下分发，那这种分发是通过什么实现的呢？我们不在每个组件中都 import store，又怎么能保证每个组件都可以调用到 store 呢？这种需求正是通过 Provider 实现的，将我们的根组件外面再嵌套一层 Provider 组件，带上 store 参数就可以了。如：

```js
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import Root from './components/Root'
import store from './store/store'

render(
  <Provider store={store}>
    <Root />
  </Provider>,
  document.getElementById('root')
)
```

假设 `<Root />` 是我们的最高组件，其它组件都嵌套在 Root 里，我们无需在每一层都传入 store，只需要在 Root 外面再嵌套一层 Provider 即可，这样 Root 所有子组件都默认可以访问到 store 中维护的 state 了。组件 Provider 的原理是 React 组件的 context 属性，感兴趣的同学可以通过阅读源码进行学习。

+ connect

React-redux 将所有的组件分成两大类，UI 组件和容器组件。

+ UI 组件

    严格的 UI 组件只负责 UI 的呈现，没有业务逻辑，没有状态，数据来源是 props，不适用 Redux 的 API

+ 容器组件

    容器组件的特征恰恰相反，它不负责 UI 的呈现，负责管理数据和业务逻辑，内部有状态，使用  Redux 的 API

React-redux 规定，UI 组件都由用户提供，容器组件由 React-redux 自动生成。connect 这个方法就是用于连接 UI 组件和容器组件，connect 其实是一个高阶函数，它输入一个 UI 组件，输出一个容器组件。我们来看一看 connect 的语法。

```js
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
```

我们着重讲解前两个参数。

+ [mapStateToProps(state, [ownProps]): stateProps] (Function)：

    如果定义这个参数，组件就会监听 Redux store 的变化。当 Redux store 发生任何变化的时候，这个方法就会被调用。这个方法必须返回一个纯对象，这个对象会和组件的 props 合并，如果我们省略这个参数，那么组件将不会监听 Redux store。如果我们指定这个方法的第二个参数 ownProps，那么只要组件接收到新的 props，这个方法也会被调用，第二个参数的值为传递到组件的 props，也就是自身的 props。

+ [mapDispatchToProps(dispatch, [ownProps]): dispatchProps] (Object or Function): 

    如果这里的参数是一个对象，那么每一个定义在该对象的函数都将被当作 Redux action creator，每个方法都会返回一个新的函数，每个新的函数里都有一个 dispatch 方法，它会将 action creator 的返回值作为执行的参数。

    如果这里的参数是一个函数，那么这个函数将会接收一个 dispatch 函数，然后我们来决定如何返回一个对象。

这也就解释了我们在组件中不显示调用 dispatch 而实现组件对 state 的交互。

## 三、实验步骤

下面我们开始编写实验中的 redux 管理 state 的部分。

### 3.1 store for home

redux 只拥有维护管理一个 store，我们先在 `/react-pxq/src` 下创建 store 文件夹

```bash
$ mkdir store
$ cd store
$ touch store.js
```

`store.js` 可以看作是我们应用所维护管理的 store，这个文件将会集合我们的一些 actions 和 reducers，再将它们作为 store 导出，供 index.js 调用。

`store.js`

```js
import {createStore} from 'redux';

let store = createStore();

export default store;
```

我们有三个组件将会使用到 redux 来管理 state，分别是 home 、production 和 record，我们分别在 `/react-pxq/src/store` 下创建对应的文件夹：

```bash
$ mkdir home production record
$ cd home
```

我们在前面的实验中已经讲解过了 redux 的基本使用方法，为了使逻辑清晰，代码容易管理，我们将 action
的 type 和 data 分开存放，分别创建文件 `action-type.js` 、`action.js`、`reducer.js`:

```bash
$ touch action-type.js action.js reducer.js
```

`/home/action-type.js`

```js
// 保存表单数据
export const SAVEFORMDATA = 'SAVEFORMDATA';

// 清空数据
export const CLEARDATA = 'CLEARDATA';
```

`/home/action.js`

```js
import * as home from './action-type';

// 保存表单数据
export const saveFormData = (value, datatype) => {
  return {
    type: home.SAVEFORMDATA,
    value,
    datatype,
  }
}

// 清除
export const clearData = () => {
  return {
    type: home.CLEARDATA,
  }
}
```

`/home/reducer.js`

```js
import * as home from './action-type';

let defaultState = {
  orderSum: '', //金额
  name: '', //姓名
  phoneNo: '', //手机号
}
// 首页表单数据
export const formData = (state = defaultState , action = {}) => {
  switch(action.type){
    case home.SAVEFORMDATA:
      return {...state, ...{[action.datatype]: action.value}};
    case home.CLEARDATA:
      return {...state, ...defaultState};
    default:
      return state;
  }
}
```

我们定义了三个事件:`SAVEFORMDATA`、`SAVEIMG`、`CLEARDATA`,home 组件上的核心内容就是表单，home 页面的表单比较复杂，既有用户的输入，又有用户通过 `/production` 页面选择的商品信息，如果我们在页面内部使用 state 处理这些逻辑，那么一定会变得十分复杂，而且不容易进行代码管理，我们通过使用 redux 统一 state 的管理。

我们将 action 的 type 和 action 的数据分开，在 action.js 中使用 `import * as home from './action-type';` 导入，在 reducer.js 中也使用同样的方式导入，在使用的时候也很方便，直接使用 `home.SAVEFORMDATA` 即可。

`reducer.js` 的编写也是比较常规的，先给出 state 的默认值，引入 action，然后使用 switch 语句，根据触发的 aaction 类型来选择返回对应的新的 state。

这是 home 组件的 store 部分，下面让我们来修改 home 组件，以达到在 home 组件中使用 redux 管理 state 的功能。

修改 `/react-pxq/src/pages/home/home.jsx` 为如下内容：

```js
import React, { Component } from 'react';
import { Link } from 'react-router-dom';
import { connect } from 'react-redux';
import { is, fromJS } from 'immutable';
import PropTypes from 'prop-types';
import { saveFormData,clearData } from '@/store/home/action';
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

  // 提交表单
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
      alertTip = '添加数据成功';
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
          </Link>
        </div>
        <div className="upload-img-con">
          <p className="common-title">请上传发票凭证</p>
          <div className="file-lable">
            <span className="common-select-btn">上传图片</span>
            <input type="file" />
          </div>
          <img src={this.props.formData.imgpath} className="select-img" alt=""/>
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
})(Home);
```

home 组件接收的 props 包含三个参数：`formData`,`saveFormData`,`clearData`:

+ formData 是一个 object 类型的参数，formData 存储了我们表单的基础数据，我们使用 formData 和其它一些数据进行拼接组成了我们用于提交的 recordData。

+ saveFormData 是放在 connect 的第二个参数里的，这个方法就是连接组件和 redux 的地方，调用这个方法后，会触发 dispatch 方法，从而触发对应的 action，调用 reducer 里对应的处理方法，改变应用的 store。当 home 组件重新渲染后，接收到的 formData 就是我们调用过 saveFormData 后的 formData。

+ clearData 同样是我们的一个 action，调用之后会调用 reducer 对应的裸机处理，清空表单数据。

```js
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
```

方法 handleInput 用于处理用户的表单输入数据，这个方法是绑定在 input 输入框的 `onchange` 事件上的，也就是说，每当用户输入信息过后，就会调用该方法，该方法的内部逻辑是先根据输入框的 type 锁定改变的数据，再对数据进行一定的规范，最后调用 `this.props.saveFormData(value,type)`,触发 action，修改 store 的内容。

```js
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
      alertTip = '添加数据成功';
      this.props.clearData();
    }
    this.setState({
      alertStatus: true,
      alertTip,
    })
  }
```

提交表单部分的内容看起来比较的复杂，其实在做的只有一件事，就是填充待提交的表单数据，因为我们提交的表单数据不单包含用户填写的内容，还包括了时间，商品佣金等需要逻辑计算的信息，所以我们声明了一个对象 `let recordData = {};`,首先将用户的填写的部分赋值，

```js
recordData.customers_name = this.props.formData.name;
recordData.phoneNo = this.props.formData.phoneNo;
```

再将创建时间、金额佣金等一些列值赋值给 recordData，因为我们提交的一定是待审核状态，所以我们把信息的状态设置为 audited， `recordData.type = "audited";`。

接着我们对 alertTip 进行赋值，因为我们不能确保用户正确填写了每一项内容。

我们在 home 中用到了 product 的一些数据，我们接下来会做讲解：

不要忘了我们的 Provider 组件，修改 `/react-pxq/src/index.js` 为如下内容：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import Route from './router/';
import {Provider} from 'react-redux';
import store from '@/store/store';
import * as serviceWorker from './serviceWorker';
import { AppContainer } from 'react-hot-loader';
import './utils/setRem';
import './style/base.css';
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

修改 `/react-pxq/src/store/store.js` 为：

```js
import {createStore, combineReducers, applyMiddleware} from 'redux';
import * as home from './home/reducer';
import thunk from 'redux-thunk';

let store = createStore(
  combineReducers({...home}),
  applyMiddleware(thunk)
);

export default store;
```

我们的项目现在并不能正常运行，因为我们还没有配置好 store.js，不要着急，我们接下来配置 production 的 store 管理部分。

### 3.2 store for production

与 store for home 类似，我们在 `/react-pxq/src/store/production` 下分别创建文件 `action-type.js` 、`action.js`、`reducer.js`:

```bash
$ touch action-type.js action.js reducer.js
```

`/production/action-type.js`

```js
// 保存商品数据
export const GETPRODUCTION = 'GETPRODUCTION';
// 选择商品
export const TOGGLESELECT = 'TOGGLESELECT';
// 编辑商品
export const EDITPRODUCTION = 'EDITPRODUCTION';
// 清空选择
export const CLEARSELECTED = 'CLEARSELECTED';
```

`/production/action.js`

```js
import * as pro from './action-type';

// 获取商品信息
export const getProData = result => {
  return {
    type:pro.GETPRODUCTION,
    dataList:result
  }
}

// 选择商品
export const togSelectPro = index => {
  return {
    type: pro.TOGGLESELECT,
    index,
  }
}

// 编辑商品
export const editPro = (index, selectNum) => {
  return {
    type: pro.EDITPRODUCTION,
    index,
    selectNum,
  }
}

// 清空选择
export const clearSelected = () => {
  return {
    type: pro.CLEARSELECTED,
  }
}
```

`/production/reducer.js`

```js
import * as pro from './action-type';
import Immutable from 'immutable';

let defaultState = {
  dataList: [],
}

export const proData = (state = defaultState, action) => {
  let imuDataList;
  let imuItem;
  switch(action.type){
    case pro.GETPRODUCTION: 
      return {...state, ...action}
    case pro.TOGGLESELECT:
      //避免引用类型数据，使用immutable进行数据转换 
      imuDataList = Immutable.List(state.dataList);
      imuItem = Immutable.Map(state.dataList[action.index]);
      imuItem = imuItem.set('selectStatus', !imuItem.get('selectStatus'));
      imuDataList = imuDataList.set(action.index, imuItem);
      // redux必须返回一个新的state
      return {...state, ...{dataList: imuDataList.toJS()}};
    case pro.EDITPRODUCTION:
      //避免引用类型数据，使用immutable进行数据转换 
      imuDataList = Immutable.List(state.dataList);
      imuItem = Immutable.Map(state.dataList[action.index]);
      imuItem = imuItem.set('selectNum', action.selectNum);
      imuDataList = imuDataList.set(action.index, imuItem);
      // redux必须返回一个新的state
      return {...state, ...{dataList: imuDataList.toJS()}};
    // 清空数据
    case pro.CLEARSELECTED:
      imuDataList = Immutable.fromJS(state.dataList);
      for (let i = 0; i < state.dataList.length; i++) {
        imuDataList = imuDataList.update(i, item => {
          item = item.set('selectStatus', false);
          item = item.set('selectNum', 0);
          return item
        })
      }
      return {...state, ...{dataList: imuDataList.toJS()}};
    default: 
      return state;
  }
}
```

以上就是 store for production 的内容，我们在 `action-type.js` 中定义了四种类型的 action，分别是 `GETPRODUCTION`,`TOGGLESELECT`,`EDITPRODUCTION`,`CLEARSELECTED`。`GETPRODUCTION` 我们暂且不做讲解，因为它涉及到我们后续的实验内容，后台 API 的部分。`TOGGLESELECT` 是当我们选中商品的时候，对应的组件的样式会发生改变，所以你可以看到在 `action.js` 中，`TOGGLESELECT` 的参数中只有 type 和 index，只传了商品的序号，只改变对应序号的样式。`EDITPRODUCTION` 是用于我们选择商品时的加减操作，有时候我们不会只选择一件商品，有时候我们可能选多了需要的商品，在 `action.js` 中，对应的方法的数据参数有两个，一个是 index，还有一个 selectNum，我们会根据 selectNum，对 index 对应的商品的数目作计算。`CLEARSELECTED` 相信就不用我再多做讲解了，清空我们选择的商品数据，在 `action.js` 中，clearSelected 方法我们只声明了 type，因为我们不需要返回一个经过计算的 state，我们只需要将每一项置空即可，所以在 `reducer.js` 中：

```js
case pro.CLEARSELECTED:
  imuDataList = Immutable.fromJS(state.dataList);
  for (let i = 0; i < state.dataList.length; i++) {
    imuDataList = imuDataList.update(i, item => {
      item = item.set('selectStatus', false);
      item = item.set('selectNum', 0);
      return item
    })
  }
```

我们使用 map() 遍历选中的商品，将它们的 `selectStatus` 和 `selectNum` 都置为初始值。然后再返回的 state 就可以看作是一个没有被操作过的 state 了。

修改 `/react-pxq/src/pages/production/production.jsx` 的代码如下：

```js
import React, { Component } from 'react';
import { is, fromJS } from 'immutable';
import { connect } from 'react-redux';
import { getProData, togSelectPro, editPro } from '@/store/production/action';
import PropTypes from 'prop-types';
import PublicHeader from '@/components/header/header';
import './production.less';

class Production extends Component{
  static propTypes = {
    proData: PropTypes.object.isRequired,
    getProData: PropTypes.func.isRequired,
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

  togSelect = index => {
    this.props.togSelectPro(index);
  }

  shouldComponentUpdate(nextProps, nextState) {
    return !is(fromJS(this.props), fromJS(nextProps)) || !is(fromJS(this.state), fromJS(nextState))
  }

  componentDidMount(){
    if(!this.props.proData.dataList.length){
      this.props.getProData();
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
  getProData,
  togSelectPro,
  editPro
})(Production);
```

我们把一些 production 对应的 action 接口添加处理，例如在选择商品的时候，我们触发 handleEdit(),当我们点击 + 的时候，参数为：

```js
this.handleEdit.bind(this, index, 1)
```

对应的意思就是 index 号商品 + selectNum = 1,同理，当我们点击 - 的时候，对应的 selectNum 为 -1。

/production/reducer.js 中的 clearSelected 是为 home 组件提供的，所以我们现在将 home.jsx 修改为以下代码：

```js
import React, { Component } from 'react';
import { Link } from 'react-router-dom';
import { connect } from 'react-redux';
import { is, fromJS } from 'immutable';
import PropTypes from 'prop-types';
import { saveFormData,clearData } from '@/store/home/action';
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
    // this.props.dispatch(saveFormData(value, type));
    this.props.saveFormData(value,type);
  }

  // 提交表单
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
          </Link>
        </div>
        <div className="upload-img-con">
          <p className="common-title">请上传发票凭证</p>
          <div className="file-lable">
            <span className="common-select-btn">上传图片</span>
            <input type="file" />
          </div>
          <img src={this.props.formData.imgpath} className="select-img" alt=""/>
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
})(Home);
```

由于 Redux 是一个 store 集合多个 reducer 的思想，我们现在有两个 reducer，我们需要把它们整合起来，然后创建一个 store，修改 `/react-pxq/src/store/store.js` 为：

```js
import {createStore, combineReducers, applyMiddleware} from 'redux';
import * as home from './home/reducer';
import * as production from './production/reducer';
import thunk from 'redux-thunk';

let store = createStore(
  combineReducers({...home, ...production}),
  applyMiddleware(thunk)
);

export default store;
```

### 3.3 store for record

同样，我们在 `/react-pxq/src/store/record` 下分别创建文件 `action-type.js` 、`action.js`、`reducer.js`:

```bash
$ touch action-type.js action.js reducer.js
```

`/record/action-type.js`

```js
// 获得记录信息
export const GETRECORD = 'GETRECORD';
```

`/record/action.js`

```js
import * as record from './action-type';

// 获得记录信息
export const getRecord = (data) => {
    return {
      type: record.GETRECORD,
      dataList:data,
    }
  }
```

`/record/reducer.js`

```js
import * as record from './action-type';

let defaultState = {
  dataList: [],
}

export const recordData = (state = defaultState, action) => {
  switch(action.type){
    case record.GETRECORD:
      console.log(action);
      return {...state, ...action}
    default: 
      return state;
  }
}
```

record 的 store 比较简单，结合 home 和 production 的部分很容易就可以理解。

修改 `recordList.jsx` 为以下代码：

```js
import React, { Component } from 'react';
import { is, fromJS } from 'immutable';
import {getRecord} from '@/store/record/action';
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
      this.props.getRecord(type);
    }
  }

  shouldComponentUpdate(nextProps, nextState){
    return !is(fromJS(this.props), fromJS(nextProps)) || !is(fromJS(this.state), fromJS(nextState))
  }

  componentWillMount(){
    let type = this.props.location.pathname.split('/')[2];
    this.props.getRecord(type);
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
  getRecord,
})(RecordList);
```

在 record/action.js 中我们其实留了一个接口：

```js
import * as record from './action-type';

// 获得记录信息
export const getRecord = (data) => {
    return {
      type: record.GETRECORD,
      dataList:data,
    }
  }

```

getRecord 的参数是 data，我们希望外部在调用这个 action 的时候，已经从后台获取到了 record 的数据，在调用这个 action 的时候，将数据作为参数传给 reducer，由 reducer 返回一个包含数据的 state。我们在后续的实验中会讲到 api 和后端部分的代码，在这里我们暂且作以上的修改。

### 3.4 store.js

最后，修改 `react-pxq/src/store/store.js` 为以下代码:

```js
import {createStore, combineReducers, applyMiddleware} from 'redux';
import * as home from './home/reducer';
import * as production from './production/reducer';
import * as record from './record/reducer';
import thunk from 'redux-thunk';

let store = createStore(
  combineReducers({...home, ...production, ...record}),
  applyMiddleware(thunk)
);

export default store;
```

## 四、总结

在本节中，我们完成了项目的 redux 数据管理，我们将整个项目的数据流使用 redux 统一了。我们使用 redux 提供的 combineReducers() 将多个 reducer 合并，然后通过使用 applyMiddleware()，我们可以实现异步的 action，触发 action 并不立即调用 reducer 处理，而是过一会才让 reducer 计算 state 的值。之后，我们将正式开始后端代码的编写。