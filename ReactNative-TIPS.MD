# 如何获取View的宽度和高度 `onLayout`

**当组件挂载或者布局变化的时候调用**，参数为：

{nativeEvent: { layout: {x, y, width, height}}}.

这个事件会当布局计算完成后立即调用一次，不过收到此事件时新的布局可能还没有在屏幕上体现，尤其是一个布局动画正在进行中的时候。

```js

  onLayoutChange(e) {
    console.log('onlayout:', e.nativeEvent.layout);
    this.setState({
      width: e.nativeEvent.layout.width,
      height: e.nativeEvent.layout.height
    });
  }


  ...

  <View style={styles.grid} onLayout={(e) => this.onLayoutChange(e)}>

```

# onPress事件如何传递自定参数


# fetch本地请求
**使用fetch调用本地请求**在安卓模拟器下必须使用机器IP地址,而不能使用"localhost"或者"127.0.0.1",否则会得到如下异常信息:
Network request failed

# windows上面启动项目时Watcher took too long to load  问题解决
(/node_modules/react-native/packager/react-packager/src/FileWatcher/index.js:102:16)修改MAX_WAIT_TIME参数

# 当父容器样式指定 `alignItems`, 子元素View的样式即使指定 `flex` 宽度也不会继承父容器宽度
如果想子元素View继承父容器的宽度，父View的样式必须去掉 `alignItmes`

# 处理渐帧动画时,比如实现连续移动使用 setNativeProps 要比 setState 方式性能要好得多

```

const Marquee = React.createClass({
  mixins: [TimerMixin],

  _previousTop: 0,
  _Styles: {},
  _ref: null,

  componentWillMount() {
    this._previousTop = 200;
    this._circleStyles = {
      style: {
        top: this._previousTop
      }
    };
  },

  _updatePosition: function () {
    this._ref && this._ref.setNativeProps(this._circleStyles);
  },

  slideUp() {

    (this._previousTop < -440 )? this._previousTop = 200 : this._previousTop -= 0.5;
    this._circleStyles.style.top = this._previousTop;
    this._updatePosition();
    this.setTimeout(
      ()=>{
        this.slideUp();
      }, 20)
  },

  componentDidMount() {
    this.slideUp()
  },

  renderItem() {
    return mockData.map((item, index) => {
      let color = (index % 2) ? 'blue' : 'green';
      return (
        <View key={index} style={{ height: 40, backgroundColor: color, width: 200}}>
          <Text>{index.toString()}</Text>
        </View>
      );
    });
  },

  render(){
    return (
      <View style={styles.marqueeContainer}>
        <View
          ref={(component) => { this._ref = component; }}
          style={styles.marquee}>
          {this.renderItem()}
        </View>
      </View>
    );
  }
});

const Marquee = React.createClass({
  mixins: [TimerMixin],

  _previousTop: 0,
  _Styles: {},
  _ref: null,

  componentWillMount() {
    this._previousTop = 200;
    this._circleStyles = {
      style: {
        top: this._previousTop
      }
    };
  },

  _updatePosition: function () {
    this._ref && this._ref.setNativeProps(this._circleStyles);
  },

  slideUp() {

    (this._previousTop < -440 )? this._previousTop = 200 : this._previousTop -= 0.5;
    this._circleStyles.style.top = this._previousTop;
    this._updatePosition();
    this.setTimeout(
      ()=>{
        this.slideUp();
      }, 20)
  },

  componentDidMount() {
    this.slideUp()
  },

  renderItem() {
    return mockData.map((item, index) => {
      let color = (index % 2) ? 'blue' : 'green';
      return (
        <View key={index} style={{ height: 40, backgroundColor: color, width: 200}}>
          <Text>{index.toString()}</Text>
        </View>
      );
    });
  },

  render(){
    return (
      <View style={styles.marqueeContainer}>
        <View
          ref={(component) => { this._ref = component; }}
          style={styles.marquee}>
          {this.renderItem()}
        </View>
      </View>
    );
  }
});

```

# IOS模拟器 变慢的问题解决

确保在ios模拟器的菜单里 `Debug->show animation` 未勾选

# 给NavigationBar配置action
配置action是不要直接指定路由，例如在wealth项目中"
```js
actionFunc={this.props.actions.routes.signIn()}
```
由于路由通过signIn进入Home，当每次Home为active时，signIn也会被重新创建，其componentDidMount也会被执行。
解决方法：通过action修改state，在app中根据状态来决定路由
```js
actionFunc={this.props.actions.deleteSessionToken}
```
在action中
```js
// 删除 sessiontoken
export function deleteSessionToken() {
  return dispatch => {
    return appAuthToken.deleteSessionToken().then(() => {
      dispatch(loggedOutState());
    });
  }
}
```
app文件
```js
shouldComponentUpdate(nextProps, nextState) {
    if (nextProps.auth.state !== this.props.auth.state) {
      if (nextProps.auth.state === LOGGED_IN_STATE) {
        this.props.actions.replace({name: 'home', tabBarName: 'tabBar', data: {}});
      } else if (nextProps.auth.state === LOGGED_OUT_STATE) {
        this.props.actions.replace({name: 'signIn', data: {}});
      } else if (nextProps.auth.state === 'sigin00') {
        // 当用户的状态是登陆之后 显示手势密码

      }
      return false;
    } else {
      return true;
    }
  }
```