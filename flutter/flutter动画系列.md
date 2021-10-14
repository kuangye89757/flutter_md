

# flutter_anim



[toc]



## 隐式动画 Animated前缀的Widget

- [x] ###### Duration:  动画时长

```dart
class _MyHomePageState extends State<MyHomePage> {
  double _height = 200;
  final colorSize = Colors.primaries.length;
  int _randomInt = 0;

  void _incrementCounter() {
    setState(() {
      _height += 50;
      if (_height > 300) _height = 200;
      _randomInt = Random().nextInt(colorSize);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: AnimatedContainer(
          duration: Duration(milliseconds: 1000),
          width: 300,
          height: _height,
          color: Colors.primaries[_randomInt],
          child: Center(
              child: Text(
            'Hi',
            style: TextStyle(fontSize: 72),
          )),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

*[仅Animated前缀修饰的widget,其属性发生变化时，会有过渡效果]()*

```dart
AnimatedContainer(duration: Duration(seconds: 1),
  width: 300,
  height: _height,
  decoration: BoxDecoration(
    gradient: LinearGradient(
      colors: [Colors.orange, Colors.white],
      begin: Alignment.bottomCenter,
      end: Alignment.topCenter,
      stops: [0.5,0.7], // 只有0.5~0.7才有渐变效果 【通过改变值达到蓄水动画】
    ),
    boxShadow: [BoxShadow(spreadRadius: 20, blurRadius: 20)],
    borderRadius: BorderRadius.circular(150), // 宽高的一半即半径，为圆
  ),
)
```

淡入淡出效果

```dart
AnimatedOpacity(
  duration: Duration(seconds: 1),
  opacity: 0.5, // 通过修改该属性
  child: Container(
    color: Colors.orange,
    width: 400,
    height: 400,
  ),
)
```

`onEnd`是动画执行结束回调，用法如下：

```dart
AnimatedAlign(
  onEnd: (){
    print('onEnd');
  },
  ...
)
```



位移效果

```dart
 AnimatedPadding(
  duration: Duration(seconds: 2),
  curve: Curves.bounceOut,
  padding: const EdgeInsets.all(8.0),// 通过修改该属性
  child: Container(
  color: Colors.blue,
  width: 300,
  height: 300,
  ),
),
```

- [x] ###### Curve: 曲线变化 https://api.flutter.dev/flutter/animation/Curves-class.html

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20210925154131577.png" alt="image-20210925154131577" align="left" style="zoom:50%;" />

- #### 多个widget切换动画 AnimatedSwitcher

```dart
AnimatedSwitcher(
  duration: Duration(seconds: 1),
    child: Center(child: CircularProgressIndicator(),),
    // child: Image.network(
    //     'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic1.zhimg.com%2Fv2-4026c1e2d7ecb2220ce5071e65bd21f4_1200x500.jpg&refer=http%3A%2F%2Fpic1.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1635145164&t=572cf8904d7de3b68b306cf1745f8ae5'
// ),
)
```

*[内部的child发生变化时，会有过渡效果]()*   **注意同一个child时，通过key来区分**

```dart
AnimatedSwitcher(
  duration: Duration(seconds: 1),
    child: Text('Hi', key: UniqueKey(), style: TextStyle(fontSize: 72),),
  	// child: Text('Hello', key: UniqueKey(), style: TextStyle(fontSize: 72),),
)
```

- transitionBuilder属性，作为补间动画的转化器  [child作用的子Widget, animation为动效内容]

支持 FadeTransition、ScaleTransition、RotationTransition等，并支持相互嵌套叠加效果

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20210925152522575.png" alt="image-20210925152522575" align="left" style="zoom:50%;" />

```dart
AnimatedSwitcher(
  transitionBuilder: (Widget child, Animation<double> animation) {
    return FadeTransition(
      opacity: animation,
      child: ScaleTransition(
        scale: animation,
        child: child,
      ),
    );
  },
  duration: Duration(seconds: 1),
    child: Text('Hello', key: UniqueKey(), style: TextStyle(fontSize: 72),),
  ),
)
```

- #### 补间动画 TweenAnimationBuilder

> tween作为补间，即填补区间/帧；那么必然需要至少两个关键帧作为参照【between就是在什么之间的意思】



***2秒内从0过渡到1，根据手机每秒频率60帧计算，中间需要补58帧***

```dart
/// 在2秒之间进行0~1的补间，过渡期间不断的call builder函数 并将过渡值通过value回传，返回每次变化的widget
TweenAnimationBuilder(
  duration: Duration(seconds: 2),
  tween: Tween(begin: 0.0, end: 1.0),
  builder: (BuildContext context, value, Widget child) {
      return Opacity(
        opacity: value,
        child: Container(
          color: Colors.blue,
          width: 300,
          height: 300,
        ),
      );
  },
)
  
tween: Tween(begin: 0.0, end: 0.3), 
tween: Tween(begin: 0.2, end: 0.8),
/// 若更改end为0.3，之后更改为0.8；则并不是每次动画从begin开始，而是从end上一次开始 
I/flutter (17115): 0.3
I/flutter (17115): 0.3047315
I/flutter (17115): 0.30946375
I/flutter (17115): 0.31373475
I/flutter (17115): 0.3180695
I/flutter (17115): 0.32258925
I/flutter (17115): 0.32688425
I/flutter (17115): 0.33115374999999997
I/flutter (17115): 0.33552899999999997
I/flutter (17115): 0.33987
I/flutter (17115): 0.3489015
I/flutter (17115): 0.353348
I/flutter (17115): 0.35789875
I/flutter (17115): 0.36238125
I/flutter (17115): 0.366988
I/flutter (17115): 0.37162925
I/flutter (17115): 0.37592
I/flutter (17115): 0.3804375
I/flutter (17115): 0.38486725
I/flutter (17115): 0.393771
I/flutter (17115): 0.3981765
I/flutter (17115): 0.40261025
I/flutter (17115): 0.4070485
I/flutter (17115): 0.41154674999999996
I/flutter (17115): 0.41604725
I/flutter (17115): 0.4206755
I/flutter (17115): 0.42962
I/flutter (17115): 0.43413749999999995
I/flutter (17115): 0.43862225
I/flutter (17115): 0.44320725
I/flutter (17115): 0.44772725
I/flutter (17115): 0.4571115
I/flutter (17115): 0.46604650000000003
I/flutter (17115): 0.47056125
I/flutter (17115): 0.47507625
I/flutter (17115): 0.47929075
I/flutter (17115): 0.48370824999999995
I/flutter (17115): 0.49270725
I/flutter (17115): 0.49726325
I/flutter (17115): 0.50157825
I/flutter (17115): 0.5060264999999999
I/flutter (17115): 0.5104715
I/flutter (17115): 0.51527775
I/flutter (17115): 0.519783
I/flutter (17115): 0.5245949999999999
I/flutter (17115): 0.53371925
I/flutter (17115): 0.5382605
I/flutter (17115): 0.54258175
I/flutter (17115): 0.54712025
I/flutter (17115): 0.55165
I/flutter (17115): 0.55619275
I/flutter (17115): 0.56622
I/flutter (17115): 0.57102725
I/flutter (17115): 0.5755835
I/flutter (17115): 0.58004375
I/flutter (17115): 0.5846335
I/flutter (17115): 0.5892347499999999
I/flutter (17115): 0.5936325
I/flutter (17115): 0.5981475
I/flutter (17115): 0.60265375
I/flutter (17115): 0.6112642500000001
I/flutter (17115): 0.61988475
I/flutter (17115): 0.6243814999999999
I/flutter (17115): 0.62889225
I/flutter (17115): 0.63314625
I/flutter (17115): 0.6375172499999999
I/flutter (17115): 0.6422055
I/flutter (17115): 0.64657925
I/flutter (17115): 0.65101125
I/flutter (17115): 0.6553835
I/flutter (17115): 0.66001725
I/flutter (17115): 0.6643405
I/flutter (17115): 0.66898825
I/flutter (17115): 0.6733945
I/flutter (17115): 0.6776800000000001
I/flutter (17115): 0.6820887499999999
I/flutter (17115): 0.68659575
I/flutter (17115): 0.69097
I/flutter (17115): 0.69547825
I/flutter (17115): 0.700089
I/flutter (17115): 0.70455175
I/flutter (17115): 0.7092185
I/flutter (17115): 0.71396675
I/flutter (17115): 0.71837275
I/flutter (17115): 0.7228367499999999
I/flutter (17115): 0.72753325
I/flutter (17115): 0.732064
I/flutter (17115): 0.7362967499999999
I/flutter (17115): 0.74082675
I/flutter (17115): 0.745355
I/flutter (17115): 0.750245
I/flutter (17115): 0.7547105000000001
I/flutter (17115): 0.7591162499999999
I/flutter (17115): 0.763619
I/flutter (17115): 0.7679549999999999
I/flutter (17115): 0.77221675
I/flutter (17115): 0.77671875
I/flutter (17115): 0.7813995
I/flutter (17115): 0.7861095
I/flutter (17115): 0.7910412499999999
I/flutter (17115): 0.795782
I/flutter (17115): 0.8
  
```

> **Tween(end: 0.3), // 由于begin只在第一次有效；后续通过更改end值达到动效；所以默认end值会赋值给begin**



- ##### Transfrom 转化

  - Transform.scale 【缩放】

  ```
  TweenAnimationBuilder(
    duration: Duration(seconds: 2),
    tween: Tween(begin: 1.0, end: 1.5),
    builder: (BuildContext context, value, Widget child) {
      return Opacity(
        opacity: 0.5,
        child: Container(
          color: Colors.blue,
          width: 300,
          height: 300,
          child: Center(
              child: Transform.scale(
                scale: value, // 缩放从1到1.5
                child: Text('Hi',style: TextStyle(fontSize: 72),),
              )
          ),
        ),
      );
    },
  )
  ```

  - Transform.rotate【旋转】

  ```
  TweenAnimationBuilder(
    duration: Duration(seconds: 2),
    tween: Tween(begin: 0.0, end: 3.14 * 2),
    builder: (BuildContext context, value, Widget child) {
      return Opacity(
        opacity: 0.5,
        child: Container(
          color: Colors.blue,
          width: 300,
          height: 300,
          child: Center(
              child: Transform.rotate(
                angle: value, // 0~2π的区间
                child: Text('Hi',style: TextStyle(fontSize: 72),),
              )
          ),
        ),
      );
    },
  )
  ```

  - Transform.translate【位移】

  ```
  TweenAnimationBuilder(
    duration: Duration(seconds: 2),
    tween: Tween(begin: Offset(0, 0), end: Offset(40, 20)),
    builder: (BuildContext context, value, Widget child) {
      return Opacity(
        opacity: 0.5,
        child: Container(
          color: Colors.blue,
          width: 300,
          height: 300,
          child: Center(
              child: Transform.translate(
                offset: value, // 相对当前坐标(0,0) 到 (40,20)
                child: Text('Hi',style: TextStyle(fontSize: 72),),
              )
          ),
        ),
      );
    },
  ),
  ```

  - 点击缩放效果，**end值在过渡过程中发生变化，会直接进行下次变化，而不会先到终点再变化**

  ```dart
  class _MyHomePageState extends State<MyHomePage> {
    bool _big = false;
  
    @override
    Widget build(BuildContext context) {
      return Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
        body: Center(
          child: TweenAnimationBuilder(
            duration: Duration(seconds: 2),
            tween: Tween(end: _big ? 172.0 : 72.0),
            builder: (BuildContext context, value, Widget child) {
              return Container(
                color: Colors.blue,
                width: 300,
                height: 300,
                child: Center(
                    child: Text('Hi',style: TextStyle(fontSize: value),)
                ),
              );
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: (){
            setState(() {
              _big = !_big;
            });
          },
          tooltip: 'Increment',
          child: Icon(Icons.add),
        ),
      );
    }
  }
  ```



### 计数器Demo

> ##### 绘制静态图，计算中间态的值及其变化区间

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211006130335595.png" alt="image-20211006130335595" style="zoom:50%; " />

```dart
Center(
  child: Container(
    width: 300,
    height: 120,
    color: Colors.blue,
    child: TweenAnimationBuilder(
      duration: Duration(seconds: 1),
      tween: Tween(begin: 7.0, end: 8.0),
      builder: (BuildContext context, value, Widget child) {
        // 3 / 2 = 1.5  ； 3 ~/ 2 = 1;
        final whole = value ~/ 1; // 取整
        final decimal = value - whole; // 取小数 【作为0~1的变化因子】
        print('value = $value, whole = $whole, decimal = $decimal');
        return Stack(
          children: [
            Positioned(
                top: -50, // 0 ~ -100 区间由中间向上到消失
                child: Text(
                  '7',
                  style: TextStyle(fontSize: 100),
                )),
            Positioned(
                top: 50,  // 100 ~ 0 区间由底部向上到中间
                child: Text(
                  '8',
                  style: TextStyle(fontSize: 100),
                )),
          ],
        );
      },
    ),
  )
)
  
// 变化区间  
I/flutter (23757): value = 7.0, whole = 7, decimal = 0.0
I/flutter (23757): value = 7.019922, whole = 7, decimal = 0.019922000000000217
I/flutter (23757): value = 7.039851, whole = 7, decimal = 0.03985099999999964
I/flutter (23757): value = 7.05976, whole = 7, decimal = 0.05975999999999981
I/flutter (23757): value = 7.073396, whole = 7, decimal = 0.0733959999999998
I/flutter (23757): value = 7.109744, whole = 7, decimal = 0.10974400000000006
I/flutter (23757): value = 7.127835, whole = 7, decimal = 0.12783500000000014
I/flutter (23757): value = 7.145244, whole = 7, decimal = 0.14524399999999993
I/flutter (23757): value = 7.182134, whole = 7, decimal = 0.18213399999999957
I/flutter (23757): value = 7.200275, whole = 7, decimal = 0.20027500000000042
I/flutter (23757): value = 7.219757, whole = 7, decimal = 0.21975700000000042
I/flutter (23757): value = 7.258955, whole = 7, decimal = 0.25895500000000027
I/flutter (23757): value = 7.295636, whole = 7, decimal = 0.295636
I/flutter (23757): value = 7.316183, whole = 7, decimal = 0.31618299999999966
I/flutter (23757): value = 7.331008, whole = 7, decimal = 0.33100799999999975
I/flutter (23757): value = 7.349326, whole = 7, decimal = 0.3493259999999996
I/flutter (23757): value = 7.368234, whole = 7, decimal = 0.3682340000000002
I/flutter (23757): value = 7.385989, whole = 7, decimal = 0.38598900000000036
I/flutter (23757): value = 7.404006, whole = 7, decimal = 0.40400599999999987
I/flutter (23757): value = 7.422675, whole = 7, decimal = 0.4226749999999999
I/flutter (23757): value = 7.441009, whole = 7, decimal = 0.4410090000000002
I/flutter (23757): value = 7.459341, whole = 7, decimal = 0.4593410000000002
I/flutter (23757): value = 7.478163, whole = 7, decimal = 0.47816300000000034
I/flutter (23757): value = 7.496413, whole = 7, decimal = 0.49641300000000044
I/flutter (23757): value = 7.514847, whole = 7, decimal = 0.5148469999999996
I/flutter (23757): value = 7.532514, whole = 7, decimal = 0.5325139999999999
I/flutter (23757): value = 7.551016, whole = 7, decimal = 0.5510159999999997
I/flutter (23757): value = 7.569222, whole = 7, decimal = 0.5692219999999999
I/flutter (23757): value = 7.585794, whole = 7, decimal = 0.5857939999999999
I/flutter (23757): value = 7.603303, whole = 7, decimal = 0.6033030000000004
I/flutter (23757): value = 7.621512, whole = 7, decimal = 0.6215120000000001
I/flutter (23757): value = 7.640065, whole = 7, decimal = 0.6400649999999999
I/flutter (23757): value = 7.657748, whole = 7, decimal = 0.6577479999999998
I/flutter (23757): value = 7.6757670000000005, whole = 7, decimal = 0.6757670000000005
I/flutter (23757): value = 7.6933489999999995, whole = 7, decimal = 0.6933489999999995
I/flutter (23757): value = 7.711219, whole = 7, decimal = 0.7112189999999998
I/flutter (23757): value = 7.728254, whole = 7, decimal = 0.7282539999999997
I/flutter (23757): value = 7.745865, whole = 7, decimal = 0.7458650000000002
I/flutter (23757): value = 7.763702, whole = 7, decimal = 0.7637020000000003
I/flutter (23757): value = 7.781576, whole = 7, decimal = 0.7815760000000003
I/flutter (23757): value = 7.799541, whole = 7, decimal = 0.7995409999999996
I/flutter (23757): value = 7.8353909999999996, whole = 7, decimal = 0.8353909999999996
I/flutter (23757): value = 7.853197, whole = 7, decimal = 0.8531969999999998
I/flutter (23757): value = 7.872091, whole = 7, decimal = 0.8720910000000002
I/flutter (23757): value = 7.890608, whole = 7, decimal = 0.8906080000000003
I/flutter (23757): value = 7.909562, whole = 7, decimal = 0.9095620000000002
I/flutter (23757): value = 7.927383, whole = 7, decimal = 0.9273829999999998
I/flutter (23757): value = 7.9671579999999995, whole = 7, decimal = 0.9671579999999995
I/flutter (23757): value = 8.0, whole = 8, decimal = 0.0
```

> ##### 根据上述变化规律，更改相应的位移因子

```dart
Stack(
  children: [
    Positioned(
        top: -100 * decimal, // 0 ~ -100 区间由中间向上到消失
        child: Text(
          '$whole',
          style: TextStyle(fontSize: 100),
        )),
    Positioned(
        top: 100 - decimal * 100,  // 100 ~ 0 区间由底部向上到中间
        child: Text(
          '${whole + 1}',
          style: TextStyle(fontSize: 100),
        )),
  ],
);

随着tween的end值变化，做到向上翻转的数字
  tween: Tween(begin: 7.0, end: 8.0), 
	tween: Tween(begin: 7.0, end: 9.0), 
```

> ##### 同样的思路，添加移动时的透明度变化

```dart
Stack(
  children: [
    Positioned(
        top: -100 * decimal, // 由中间向上到消失 0 ~ -100
        child: Opacity(
          opacity: 1.0 - decimal, // 由中间向上到消失 1.0 ~ 0.0
          child: Text(
            '$whole',
            style: TextStyle(fontSize: 100),
          ),
        )),
    Positioned(
        top: 100 - decimal * 100,  // 由底部向上到中间 100 ~ 0
        child: Opacity(
          opacity: decimal, // 由底部向上到中间 0.0 ~ 1.0
          child: Text(
            '${whole + 1}',
            style: TextStyle(fontSize: 100),
          ),
        )),
  ],
);

随着tween的end值变化，做到向上/向下翻转的数字
  tween: Tween(begin: 7.0, end: 8.0), 
	tween: Tween(begin: 7.0, end: 9.0), 
	tween: Tween(begin: 7.0, end: 1.0), 
```



### 完整实例

```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  _increment(){
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Container(
          width: 300,
          height: 120,
          child: Row(
          mainAxisAlignment: MainAxisAlignment.start,
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            AnimatedCounter(
              width: 80,
              duration: Duration(seconds: 1),
              value: 1,
            ),
            AnimatedCounter(
              width: 80,
              duration: Duration(seconds: 1),
              value: 0,
            ),
            AnimatedCounter(
              width: 80,
              duration: Duration(seconds: 1),
              value: _counter,
            ),
          ],
        ),
      )
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _increment,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}



class AnimatedCounter extends StatelessWidget{

  final int value;
  final Duration duration;
  final double width;

  AnimatedCounter({this.width, @required this.value, @required this.duration});

  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder(
      duration: duration,
      tween: Tween(end: value.toDouble()),
      builder: (BuildContext context, value, Widget child) {
        final whole = value ~/ 1; // 取整
        final decimal = value - whole; // 取小数 【作为0~1的变化因子】
        return Container(
          width: width,
          child: Stack(
            children: [
              Positioned(
                  top: -100 * decimal, // 由中间向上到消失 0 ~ -100
                  child: Opacity(
                    opacity: 1.0 - decimal, // 由中间向上到消失 1.0 ~ 0.0
                    child: Text(
                      '$whole',
                      style: TextStyle(fontSize: 100),
                    ),
                  )),
              Positioned(
                  top: 100 - decimal * 100,  // 由底部向上到中间 100 ~ 0
                  child: Opacity(
                    opacity: decimal, // 由底部向上到中间 0.0 ~ 1.0
                    child: Text(
                      '${whole + 1}',
                      style: TextStyle(fontSize: 100),
                    ),
                  )),
            ],
          ),
        );
      },
    );
  }

}
```

## 显式动画 Transtion后缀结尾

> 完全手动控制，所以不再默认提供duration; 而是**AnimationController**



### SingleTickerProviderStateMixin

> single tick即 每次滴答一下就进行一次回调



### AnimationController

- duration：动画时长，根据时长计算垂直同步的频率

- vsync垂直同步
  - 屏幕刷新频率
  - 有些手机需要16ms、有些手机需要8ms等等，跟机型有关
  - 通过提供好的mixin设置即可  [SingleTickerProviderStateMixin]()
- 这样每次屏幕刷新就会收到回调，进而根据情况来更新，形成动画

```dart
AnimationController _controller;

@override
void initState() {
  _controller = AnimationController(
    duration: Duration(seconds: 2),
    vsync: this
  );
  _controller.addListener(() {
    print(_controller.value); // 监听2s内帧数的变化
  });
  super.initState();
}

@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

_controller.forward(); // 向前执行
_controller.repeat(); // 不停执行
_controller.repeat(reverse: true); // 不停执行 [toggle效果]
_controller.stop(); // 原地停止
_controller.reset(); // 停止执行，并重置

// 默认在0~1之间变化 
_controller = AnimationController(
  duration: Duration(seconds: 2),
  lowerBound: 3.0,
  upperBound: 5.0,
  vsync: this
);

I/flutter (24546): 3.0
I/flutter (24546): 3.05335
I/flutter (24546): 3.071114
I/flutter (24546): 3.071114
I/flutter (24546): 3.090062
I/flutter (24546): 3.108324
I/flutter (24546): 3.1260540000000003
I/flutter (24546): 3.179748
I/flutter (24546): 3.1972050000000003
I/flutter (24546): 3.214655
I/flutter (24546): 3.233654
I/flutter (24546): 3.251118
I/flutter (24546): 3.269018
I/flutter (24546): 3.2875300000000003
I/flutter (24546): 3.304984
I/flutter (24546): 3.3238820000000002
I/flutter (24546): 3.340425
I/flutter (24546): 3.358508
I/flutter (24546): 3.376147
I/flutter (24546): 3.392987
I/flutter (24546): 3.410385
I/flutter (24546): 3.42862
I/flutter (24546): 3.446288
I/flutter (24546): 3.462716
I/flutter (24546): 3.479741
I/flutter (24546): 3.496889
I/flutter (24546): 3.514565
I/flutter (24546): 3.533083
I/flutter (24546): 3.550751
I/flutter (24546): 3.568737
I/flutter (24546): 3.586512
I/flutter (24546): 3.605358
I/flutter (24546): 3.622972
I/flutter (24546): 3.640757
I/flutter (24546): 3.658989
I/flutter (24546): 3.67643
I/flutter (24546): 3.694105
I/flutter (24546): 3.711777
I/flutter (24546): 3.730449
I/flutter (24546): 3.7475750000000003
I/flutter (24546): 3.7652390000000002
I/flutter (24546): 3.7830529999999998
I/flutter (24546): 3.801464
I/flutter (24546): 3.8192180000000002
I/flutter (24546): 3.8369720000000003
I/flutter (24546): 3.856108
I/flutter (24546): 3.874129
I/flutter (24546): 3.892131
I/flutter (24546): 3.90991
I/flutter (24546): 3.928
I/flutter (24546): 3.945917
I/flutter (24546): 3.9632389999999997
I/flutter (24546): 3.980514
I/flutter (24546): 3.997495
I/flutter (24546): 4.014811
I/flutter (24546): 4.03266
I/flutter (24546): 4.050182
I/flutter (24546): 4.067657
I/flutter (24546): 4.085138
I/flutter (24546): 4.103675
I/flutter (24546): 4.12116
I/flutter (24546): 4.139277
I/flutter (24546): 4.156845
I/flutter (24546): 4.174325
I/flutter (24546): 4.192506
I/flutter (24546): 4.2103909999999996
I/flutter (24546): 4.228118
I/flutter (24546): 4.246096
I/flutter (24546): 4.263821
I/flutter (24546): 4.28231
I/flutter (24546): 4.30089
I/flutter (24546): 4.320223
I/flutter (24546): 4.355213
I/flutter (24546): 4.373493
I/flutter (24546): 4.391583
I/flutter (24546): 4.409564
I/flutter (24546): 4.426583
I/flutter (24546): 4.445336
I/flutter (24546): 4.461622
I/flutter (24546): 4.479627
I/flutter (24546): 4.497917
I/flutter (24546): 4.5154440000000005
I/flutter (24546): 4.533445
I/flutter (24546): 4.551643
I/flutter (24546): 4.569642
I/flutter (24546): 4.586722
I/flutter (24546): 4.604583
I/flutter (24546): 4.622197
I/flutter (24546): 4.64026
I/flutter (24546): 4.657674
I/flutter (24546): 4.67562
I/flutter (24546): 4.693690999999999
I/flutter (24546): 4.710672
I/flutter (24546): 4.728343
I/flutter (24546): 4.746188
I/flutter (24546): 4.764994
I/flutter (24546): 4.782673
I/flutter (24546): 4.800447
I/flutter (24546): 4.818274
I/flutter (24546): 4.835261
I/flutter (24546): 4.853085999999999
I/flutter (24546): 4.871053
I/flutter (24546): 4.889293
I/flutter (24546): 4.906695
I/flutter (24546): 4.924382
I/flutter (24546): 4.9421479999999995
I/flutter (24546): 4.959422
I/flutter (24546): 4.977194
I/flutter (24546): 4.994856
I/flutter (24546): 5.0


```

> 旋转动画

```dart
// class AnimationController extends Animation<double>
即一系列的小数变化控制器 【从lowerBound到upperBound】
  
RotationTransition(
  turns: _controller, // 接收一个Animation<double>， AnimationController即可
  child: Container(
    width: 300,
    height: 300,
    color: Colors.blue,
  ),
)),
```

- **_controller.drive(Tween(begin: 0.5, end: 2.0)),**

- **Tween(begin: 0.5, end: 2.0).animate(_controller),**

> 上述两种写法意思一致。都是将AnimationController的value，映射成Tween

```
ScaleTransition(
  scale: _controller.drive(Tween(begin: 0.5, end: 2.0)),
  child: Container(
    width: 300,
    height: 300,
    color: Colors.blue,
  ),
)

这样 最小时就是0.5，最大为2.0；因子数会根据其进行映射变化  此时_controller.value的区间还是lowerBound到upperBound
但实际scale接收到的Animation区间在0.5~2.0

```

**这样映射的好处是，若接收的是一个Animation<非double>则可以通过Tween去设置区间，再由controller驱动**

*Offset：0.0为原位置，0.5为X,Y的半个身位置，正负代表方向*

```dart
_controller = AnimationController(
  duration: Duration(seconds: 5),
  vsync: this
);

SlideTransition(
  position: Tween(begin: Offset(0.0, 0.0), end: Offset(0.0, 0.5))
      .chain(CurveTween(curve: Curves.elasticOut)) // 支持嵌套Tween
  		.chain(CurveTween(curve: Interval(0.8, 1.0))) // 时长的最后20%，即4~5秒之间才开始执行动画
      .animate(_controller),
  child: Container(
    width: 300,
    height: 300,
    color: Colors.blue,
  ),
))
```

嵌套的模式，相当于函数间嵌套 h(g(f(x)))  



### 交错动画  Tween + Interval

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211006161452825.png" alt="image-20211006161452825" style="zoom:50%;" />

```
Center(
    child: Column(
  mainAxisAlignment: MainAxisAlignment.center,
  children: [
    SlideBox(controller: _controller, color: Colors.blue[200], interval: Interval(0.0, 0.2)),
    SlideBox(controller: _controller, color: Colors.blue[400], interval: Interval(0.2, 0.4)),
    SlideBox(controller: _controller, color: Colors.blue[600], interval: Interval(0.4, 0.6)),
    SlideBox(controller: _controller, color: Colors.blue[800], interval: Interval(0.6, 0.8)),
    SlideBox(controller: _controller, color: Colors.blue, interval: Interval(0.8, 1.0)),
  ],
)),

class SlideBox extends StatelessWidget {
  final AnimationController controller;
  final Interval interval;
  final Color color;

  SlideBox({
    Key key,
    @required this.controller,
    @required this.interval,
    @required this.color,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      position: Tween(begin: Offset.zero, end: Offset(0.2, 0.0))
          .chain(CurveTween(curve: Curves.bounceOut))
          .chain(CurveTween(curve: interval))
          .animate(controller),
      child: Container(
        width: 300,
        height: 100,
        color: color,
      ),
    );
  }
}
```



### AnimationBuilder

> 自定义手动动画

- animation :  接收一个AnimationController，用于监听
- builder ： 监听AnimationController的值；每当发生变化会回调

```dart
AnimatedBuilder(
  animation: _controller,
  builder: (BuildContext context, Widget child) {
    // 这里的child 即是AnimatedBuilder的child属性传入的，这样防止每次调用build都重新创建
    return Opacity(
      opacity: _controller.value,
      child: Container(
        width: 300,
        height: 200 + 100 * _controller.value,
        color: Colors.blue,
        child: child,
      ),
    );
  },
  child: Center(
    child: Text(
      'Hi',
      style: TextStyle(fontSize: 72),
    ),
  ),
)
```

- 这样child可以处理不受动画控制的部分的widget, **build回调时不会做渲染计算**；以达到优化效果

#### 方式一：

- 通过**Tween的evaluate(_controller)**来评估值的区间，这样就能利用AnimationController的值做任何渐变的变化

```dart
AnimatedBuilder(
  animation: _controller,
  builder: (BuildContext context, Widget child) {
    return Opacity(
    	// opacity: _controller.value,
      opacity: Tween(begin: 0.5, end: 0.8).evaluate(_controller),
      child: Container(
        width: 300,
        // height: 200 + 100 * _controller.value,
        height: Tween(begin: 200.0, end: 300.0).evaluate(_controller),
        color: Colors.blue,
        child: child,
      ),
    );
  },
  child: Center(
    child: Text(
      'Hi',
      style: TextStyle(fontSize: 72),
    ),
  ),
)
```

#### 方式二：

- 通过**定义多个Tween的animate**来得到Animation，直接获取Animation的值做任何渐变的变化

```dart
final Animation opacityAnimation = Tween(begin: 0.5, end: 0.8).animate(_controller);
final Animation heightAnimation =
        Tween(begin: 200.0, end: 300.0)
            .chain(CurveTween(curve: Curves.bounceOut))
            .animate(_controller);

AnimatedBuilder(
   animation: _controller, // 注意_controller要一致
   builder: (BuildContext context, Widget child) {
     return Opacity(
        opacity: opacityAnimation.value,
        child: Container(
          width: 300,
          height: heightAnimation.value,
          color: Colors.blue,
          child: child,
        )
     );
  },
  child: Center(
  	child: Text('Hi', style: TextStyle(fontSize: 72)),
  ),
);
```



### 4-6-7呼吸动画

- 编写widget，设置动画关键帧时候的样式，以确定动画区间

```
Container(
  width: 300,
  height: 300,
  decoration: BoxDecoration(
    color: Colors.blue,
    shape: BoxShape.circle,
    gradient: RadialGradient(
      colors: [
        Colors.blue[600],
        Colors.blue[100],
      ],
      stops: [
        0.5,
        0.6
      ], // 0~0.4之间blue[600]， 0.5~0.6之间渐变 0.6~1.0之间blue[400]
    ),
  ),
)
```

- 使用AnimatedBuilder设置动画

```dart
/// 显式动画页
class TransitionPage extends StatefulWidget {
  TransitionPage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _TransitionPageState createState() => _TransitionPageState();
}

class _TransitionPageState extends State<TransitionPage>
    with SingleTickerProviderStateMixin {

  AnimationController _controller;

  @override
  void initState() {
    _controller = AnimationController(
      duration: Duration(seconds: 4),
      vsync: this,
    )..repeat(reverse: true);
    super.initState();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
          child: AnimatedBuilder(
        animation: _controller,
        builder: (BuildContext context, Widget child) {
          return Container(
            width: 300,
            height: 300,
            decoration: BoxDecoration(
              color: Colors.blue,
              shape: BoxShape.circle,
              gradient: RadialGradient(
                colors: [
                  Colors.blue[600],
                  Colors.blue[100],
                ],
                stops: [
                  _controller.value,
                  _controller.value + 0.1
                ], 
              ),
            ),
          );
        },
      )),
    );
  }
}
```

> **方式一：使用交错动画 Tween + Interval方式**

```dart
Widget build(BuildContext context) {
  /// 吸吸吸吸(20%) 停停停停停停停(35%) 呼呼呼呼呼呼呼呼(95%) 停 【478呼吸法时长 20秒】
  Animation animation1 = Tween(begin: 0.0, end: 1.0)
      .chain(CurveTween(curve: Interval(0.0, 0.2)))
      .animate(_controller);

  Animation animation3 = Tween(begin: 1.0, end: 0.0)
      .chain(CurveTween(curve: Interval(0.4, 0.95)))
      .animate(_controller);

  return Scaffold(
    appBar: AppBar(
      title: Text(widget.title),
    ),
    body: Center(
        child: AnimatedBuilder(
      animation: _controller,
      builder: (BuildContext context, Widget child) {
        return Container(
          width: 300,
          height: 300,
          decoration: BoxDecoration(
            color: Colors.blue,
            shape: BoxShape.circle,
            gradient: RadialGradient(
              colors: [
                Colors.blue[600],
                Colors.blue[100],
              ],
              stops: _controller.value <= 0.2
                  ? [animation1.value, animation1.value + 0.1]
                  : [animation3.value, animation3.value + 0.1],
            ),
          ),
        );
      },
    )),
  );
}
```

> **方式二：使用两个AnimationController + Future/await方式**

**PS: 多个AnimationController需要mixin [TickerProviderStateMixin]()**

```dart
class TransitionPage extends StatefulWidget {
  TransitionPage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _TransitionPageState createState() => _TransitionPageState();
}

class _TransitionPageState extends State<TransitionPage>
    with TickerProviderStateMixin {

  AnimationController _expansionController;
  AnimationController _opacityController;

  @override
  void initState() {
    _expansionController = AnimationController(vsync: this);
    _opacityController = AnimationController(vsync: this);
    super.initState();
  }

  @override
  void dispose() {
    _expansionController.dispose();
    _opacityController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
          child: FadeTransition(
            opacity: Tween(begin: 1.0, end: 0.5).animate(_opacityController),
            child: AnimatedBuilder(
        animation: _expansionController,
        builder: (BuildContext context, Widget child) {
            return Container(
              width: 300,
              height: 300,
              decoration: BoxDecoration(
                color: Colors.blue,
                shape: BoxShape.circle,
                gradient: RadialGradient(
                  colors: [
                    Colors.blue[600],
                    Colors.blue[100],
                  ],
                  stops: [_expansionController.value, _expansionController.value + 0.1],
                ),
              ),
            );
        },
      ),
          ),),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          /// 吸吸吸吸(4s) 停停停停停停停(7s) 呼呼呼呼呼呼呼呼(8s) 停 【478呼吸法时长 20秒】
          _expansionController.duration = Duration(seconds: 4);
          _expansionController.forward();
          await Future.delayed(Duration(seconds: 4));

          /// fade两次 7000 / 4
          _opacityController.duration = Duration(milliseconds: 1750);
          _opacityController.repeat(reverse: true);
          await Future.delayed(Duration(seconds: 7));
          _opacityController.reset();

          _expansionController.duration = Duration(seconds: 8);
          _expansionController.reverse();
        },
        child: Icon(Icons.add),
      ),
    );


  }
}
```



### AlignTransition

对Align子控件位置变换动画，用法如下：

```dart
@override
  void initState() {
    _animationController =
        AnimationController(duration: Duration(seconds: 2), vsync: this);
    _animation = Tween<AlignmentGeometry>(
            begin: Alignment.topLeft, end: Alignment.bottomRight)
        .animate(_animationController);

    //开始动画
    _animationController.forward();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 200,
      width: 200,
      color: Colors.blue,
      child: AlignTransition(
        alignment: _animation,
        child: Container(
          height: 30,
          width: 30,
          color: Colors.red,
        ),
      ),
    );
  }
```

效果如下：

![](/Users/shijiewang/Documents/my_resources/inke/md/widgets/img/AlignTransition/AlignTransition_1.gif)







## 动画原理

### ImplicitlyAnimatedWidget -- 隐式动画组件

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211007175800923.png" alt="image-20211007175800923" style="zoom:50%;" />

**从源码中可以看出，其实只是封装了AnimationController，方便使用而已**

```
abstract class ImplicitlyAnimatedWidgetState<T extends ImplicitlyAnimatedWidget> extends State<T> 
			with SingleTickerProviderStateMixin<T> {

	@protected
  AnimationController get controller => _controller;
  AnimationController _controller;

  /// The animation driving this widget's implicit animations.
  Animation<double> get animation => _animation;
  Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.duration,
      debugLabel: kDebugMode ? widget.toStringShort() : null,
      vsync: this,
    );
    _controller.addStatusListener((AnimationStatus status) {
      switch (status) {
        case AnimationStatus.completed:
          if (widget.onEnd != null)
            widget.onEnd();
          break;
        case AnimationStatus.dismissed:
        case AnimationStatus.forward:
        case AnimationStatus.reverse:
      }
    });
    _updateCurve();
    _constructTweens();
    didUpdateTweens();
  }
```

### AnimatedWidget -- 显式手动控制动画组件

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211007180242406.png" alt="image-20211007180242406" style="zoom:50%;" />

**传入的animation属性，其实就是一个监听动画变化的监听器listenable**

```dart
class _AnimatedState extends State<AnimatedWidget> {
  @override
  void initState() {
    super.initState();
    widget.listenable.addListener(_handleChange);
  }

  @override
  void didUpdateWidget(AnimatedWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.listenable != oldWidget.listenable) {
      oldWidget.listenable.removeListener(_handleChange);
      widget.listenable.addListener(_handleChange);
    }
  }

  @override
  void dispose() {
    widget.listenable.removeListener(_handleChange);
    super.dispose();
  }
  ...
    
  void _handleChange() {
    setState(() {
      // The listenable's state is our build state, and it changed already.
    });
  }
} 
```

- 每当值发生变化的时候，执行*_handleChange* 直接调setState
- 由于flutter引擎对渲染控件优化的很极致，既是多次build也不用担心，1秒内60或120次的setState也不会造成影响



### Ticker

> 真正执行_handleChange的触发器 【Ticker + setState 】

```dart
typedef TickerCallback = void Function(Duration elapsed);

final TickerCallback _onTick;
Ticker(this._onTick, { this.debugLabel }) {
  assert(() {
    _debugCreationStack = StackTrace.current;
    return true;
  }());
}
```

创建一个Ticker，启动之后就能不断的接收到绘制帧的垂直同步信号Vsync，之后就能来更新UI

```
@override
void initState() {
  Ticker ticker = Ticker((elapsed){
    print('ticker : $elapsed');
  });
  ticker.start();
  super.initState();
}


I/flutter (  338): ticker : 0:00:00.000000
I/flutter (  338): ticker : 0:00:00.018553
I/flutter (  338): ticker : 0:00:00.037101
I/flutter (  338): ticker : 0:00:00.055644
I/flutter (  338): ticker : 0:00:00.077122
I/flutter (  338): ticker : 0:00:00.095821
I/flutter (  338): ticker : 0:00:00.114381
I/flutter (  338): ticker : 0:00:00.132707
```

### TickerProviderStateMixin

> 内部维护了一个Ticker集合，通过实现TickerProvider，重写createTicker来创建一个Ticker集合；帮助管理动画变化

### SingleTickerProviderStateMixin

> 同上，不过只能维护一个Ticker对象，这也就是为什么只有一个AnimationController的时候用这个



```
以上，通过Ticker来触发和管理动画变化，将细节实现隐藏帮助我们处理边界和生命周期管理的场景；

同时，由于机型不同，刷新频率不同；如果直接使用Ticker+setState需要考虑诸多因素

所以无需直接使用Ticker
```



## 主动画 Hero

> **可以在路由(页面)之间“飞行”的widget**，即在两个的页面都有的控件，建立关联的动画，默认会自动找到一条曲线平滑过渡

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211007191854198.png" alt="image-20211007191854198" style="zoom:50%;" /><img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211007192018024.png" alt="image-20211007192018024" style="zoom:50%;" />

#### 使用同一个tag

> 触发后过渡期间，会***立即转变成目标widget***, 若两个widget类型相同则可以看出过渡效果
>
> **所以一定要设置相同widget, 否则会很奇怪**

列表页

```
const path = 'assets/images/pic.webp';

class HeroDemoPage extends StatefulWidget {
  HeroDemoPage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _HeroDemoPageState createState() => _HeroDemoPageState();
}

class _HeroDemoPageState extends State<HeroDemoPage> {

  @override
  void initState() {
    // 动画效果放慢5倍，用于测试
    timeDilation = 5.0;
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
      body: GridView.count(
        crossAxisCount: 3,
        crossAxisSpacing: 2.0,
        mainAxisSpacing: 2.0,
        children: List.generate(15, (index) {
          final tag = '${path}_$index';
          return GestureDetector(onTap: (){
            Navigator.push(context, MaterialPageRoute(builder: (_){
                return DetailScreen(path, tag);
            }));
            },
            child: Hero(
              tag: tag,
              child: Image.asset(path),
            ),
          );
        }),
      ),
    );
  }
}
```

详情页

```
class DetailScreen extends StatefulWidget {
  final String path;
  final String tag;
  const DetailScreen(this.path, this.tag, {Key key}) : super(key: key);

  @override
  _DetailScreenState createState() => _DetailScreenState();
}

class _DetailScreenState extends State<DetailScreen> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: GestureDetector(
        onTap: (){
          Navigator.pop(context);
        },
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Container(
              width: 400,
              height: 400,
              child: Hero(
                tag: '${widget.tag}',
                child: Image.asset(
                  widget.path,
                  width: 400,
                  height: 400,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Text(
                'Lorem ipsum dolor sit amet',
                style: TextStyle(fontSize: 24),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## CustomPainter

> 直接操作底层来绘制动画 【类似Android的自定义View的onDraw方法】

使用`CustomPaint`控件，传入具体的画笔`CustomPainter`，来操作底层

```
class _CustomerPainterPageState extends State<CustomerPainterPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: SafeArea(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.blue,
          child: CustomPaint(
            painter: MyPainter(),
          ),
        ),
      ),
    );
  }
}


class MyPainter extends CustomPainter{
  
  @override
  void paint(Canvas canvas, Size size) {
    /// size = Size(300.0, 300.0)
    print('size = $size');
  }

	/// 通过传入上一帧的CustomPainter，来判断是否需要重绘，作为演示始终设置为true
  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;

}
```

**CustomPainter 需要实现2个函数**：

- `paint(Canvas canvas, Size size)`

  - **canvas** : 画布，提供绘制图形的API
  - **size**：父类约束的大小尺寸

- `shouldRepaint(covariant CustomPainter oldDelegate)`

  - 通过传入上一帧的CustomPainter，来判断**是否需要重绘**，作为演示始终设置为true

  - 配合AnimatedBuilder, 则每次AnimationController发生变化，回调build函数，触发CustomPainter的shouldRepaint进行判断

  - ```dart
    AnimatedBuilder(
      animation: _animationController,
      builder: (_, __){
        return CustomPaint(
          painter: MyPainter(),
        );
      },
    )
    ```

  

### Canvas

> 画布，提供绘制图形的API

一、绘制圆

```dart
canvas.drawCircle(size.center(Offset.zero), 20, Paint()); // 画布中心位置绘制圆，半径20
```

***size.center***

```dart
// 当前尺寸的中心点位置，参数为相当于当前中心位置的偏移
Offset center(Offset origin) => Offset(origin.dx + width / 2.0, origin.dy + height / 2.0);
```

二、绘制椭圆、绘制矩形

```dart
canvas.drawOval(Rect.fromCenter(
  center: size.center(Offset(0,250)),
  width: 200,
  height: 250
), Paint());

canvas.drawRect(Rect.fromCenter(
      center: size.center(Offset(0,250)),
      width: 200,
      height: 250
), Paint());
```



### Paint

> 画笔工具 

```dart
final paint = Paint()..color = Colors.white; // 白色画笔
```



### 完整演示

```dart
class CustomerPainterPage extends StatefulWidget {
  CustomerPainterPage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _CustomerPainterPageState createState() => _CustomerPainterPageState();
}

class _CustomerPainterPageState extends State<CustomerPainterPage>
    with SingleTickerProviderStateMixin {
  AnimationController _animationController;
  List<Snowflake> _snowflakes = List.generate(600, (index) => Snowflake());

  @override
  void initState() {
    _animationController =
        AnimationController(duration: Duration(seconds: 1), vsync: this)
          ..repeat();
    super.initState();
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: SafeArea(
        child: Container(
          constraints: BoxConstraints.expand(),
          decoration: BoxDecoration(
            gradient: LinearGradient(
              begin: Alignment.topCenter,
              end: Alignment.bottomCenter,
              colors: [
                Colors.blue,
                Colors.lightBlue,
                Colors.white,
              ],
              stops: [0.0, 0.8, 0.95],
            ),
          ),
          child: AnimatedBuilder(
            animation: _animationController,
            builder: (_, __) {
              // 雪花下落
              _snowflakes.forEach((snowflake) => snowflake.fail());
              return CustomPaint(
                painter: MyPainter(_snowflakes),
              );
            },
          ),
        ),
      ),
    );
  }
}

class MyPainter extends CustomPainter {
  final List<Snowflake> _snowflakes;

  MyPainter(this._snowflakes);

  /// canvas : 画笔，提供绘制图形的API
  /// size：父类约束的大小尺寸
  @override
  void paint(Canvas canvas, Size size) {
    /// size = Size(300.0, 300.0)
    print('size = $size');

    /// 画笔
    final whitePaint = Paint()..color = Colors.white;

    /// 雪人头
    canvas.drawCircle(size.center(Offset(0, 90)), 60, whitePaint);

    /// 雪人身体
    canvas.drawOval(
        Rect.fromCenter(
            center: size.center(Offset(0, 250)), width: 200, height: 250),
        whitePaint);

    /// 雪花
    _snowflakes.forEach((snowflake) {
      canvas.drawCircle(
          Offset(snowflake.x, snowflake.y), snowflake.radius, whitePaint);
    });
  }

  /// 通过传入上一帧的CustomPainter，来判断是否需要重绘，作为演示始终设置为true
  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}

/// 雪花
class Snowflake {
  // 屏幕宽度随机位置
  double x = Random().nextDouble() * 400;

  // 屏幕高度随机位置
  double y = Random().nextDouble() * 700;

  // 2~4之间雪花大小
  double radius = Random().nextDouble() * 2 + 2;

  // 2~6之间雪花下落速度
  double velocity = Random().nextDouble() * 4 + 2;

  fail() {
    y += velocity;
    if (y > 700) {
      // 超出屏幕，则重置
      x = Random().nextDouble() * 400;
      y = 0;
      radius = Random().nextDouble() * 2 + 2;
      velocity = Random().nextDouble() * 4 + 2;
    }
  }
}
```
