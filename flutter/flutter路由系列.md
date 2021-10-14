# 路由系列



[toc]

## 监听路由堆栈变化



- **navigatorObservers(路由监听列表)**
  - ***MaterialApp***的一个属性
  - 接收一个**RouteObserver**数组



### RouteObserver(路由监听)

```
final RouteObserver<PageRoute> routeObserver = RouteObserver<PageRoute>();
```

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211009170325670.png" alt="image-20211009170325670" style="zoom:50%;" />

- 继承自**NavigatorObserver**，监听路由的入栈、出栈、替换、删除等操作
- 通过`RouteAware`接口回调通知状态变化
- 接收参数`Route`

> **仿照RouteObserver写一个自定义的路由监听**

```dart
class CustomRouteObserver<R extends Route<dynamic>> extends NavigatorObserver {
  final Map<R, Set<RouteAware>> _listeners = <R, Set<RouteAware>>{};

  void subscribe(RouteAware routeAware, R route) {
    assert(routeAware != null);
    assert(route != null);
    final Set<RouteAware> subscribers =
        _listeners.putIfAbsent(route, () => <RouteAware>{});
    if (subscribers.add(routeAware)) {
      routeAware.didPush(); /// 注册时调用
    }
  }

  void unsubscribe(RouteAware routeAware) {
    assert(routeAware != null);
    for (final R route in _listeners.keys) {
      final Set<RouteAware> subscribers = _listeners[route];
      subscribers?.remove(routeAware);
    }
  }

  @override
  void didPop(Route<dynamic> route, Route<dynamic> previousRoute) {
    if (route is R && previousRoute is R) {
      /// 先调用 【前页面】 的didPopNext
      final List<RouteAware> previousSubscribers =
          _listeners[previousRoute]?.toList();

      if (previousSubscribers != null) {
        for (final RouteAware routeAware in previousSubscribers) {
          routeAware.didPopNext();
        }
      }

      /// 先调用 【当前页面】 的didPop
      final List<RouteAware> subscribers = _listeners[route]?.toList();
      if (subscribers != null) {
        for (final RouteAware routeAware in subscribers) {
          routeAware.didPop();
        }
      }
    }
  }

  @override
  void didPush(Route<dynamic> route, Route<dynamic> previousRoute) {
    if (route is R && previousRoute is R) {
      /// 仅调用 【前页面】 的didPushNext
      final Set<RouteAware> previousSubscribers = _listeners[previousRoute];
      if (previousSubscribers != null) {
        for (final RouteAware routeAware in previousSubscribers) {
          routeAware.didPushNext();
        }
      }
    }
  }

  @override
  void didRemove(Route route, Route previousRoute) {
    /// 
    print('previousRoute : ${previousRoute.settings.name}');
    print('route : ${route.settings.name}');
  }

  @override
  void didReplace({Route newRoute, Route oldRoute}) {
    /// pushReplacement时调用
    print('oldRoute : ${oldRoute.settings.name}');
    print('newRoute : ${newRoute.settings.name}');
  }

}
```

### 编写Demo

> 仅编写ARouterObserverDemo，其他同理

```dart
class FakeApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      routes: <String, WidgetBuilder> {
        '/ARouteObserver': (context) => ARouterObserverDemo(),
        '/BRouteObserver': (context) => BRouterObserverDemo(),
        '/CRouteObserver': (context) => CRouterObserverDemo(),
      },
      navigatorObservers: [routeObserver],
      initialRoute: '/ARouteObserver',
    );
  }
}

/// ARouterObserverDemo 
class _ARouterObserverDemoState extends State<ARouterObserverDemo> with RouteAware {

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    routeObserver.subscribe(this, ModalRoute.of(context));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        alignment: Alignment.center,
        child: RaisedButton(
          child: Text('A RouteObserver'),
          onPressed: (){
            Navigator.of(context).pushNamed('/BRouteObserver');
          },
        ),
      ),
    );
  }

  @override
  void dispose() {
    super.dispose();
    routeObserver.unsubscribe(this);
  }

  @override
  void didPush() {
    final router = ModalRoute.of(context).settings.name;
    print('A-didPush route: $router');
  }

  @override
  void didPop() {
    final router = ModalRoute.of(context).settings.name;
    print('A-didPop route: $router');
  }

  @override
  void didPushNext() {
    final router = ModalRoute.of(context).settings.name;
    print('A-didPushNext route: $router');
  }

  @override
  void didPopNext() {
    final router = ModalRoute.of(context).settings.name;
    print('A-didPopNext route: $router');
  }
}

```

**主要复写了来自`RouteAware`的 didPush、didPop、didPushNext、didPopNext四个回调函数**

从`RouteObserver`的源码中可以得知：

- didPush：在`RouteAware`**注册**时调用
- didPop: 在`RouteAware`**出栈时当前页**调用
- didPushNext：在`RouteAware`**入栈**时调用
- didPopNext：在`RouteAware`**出栈时上一个页**调用



---

### 演示

```
启动进入A页

2021-10-09 16:45:02.908 1947-2171/com.xunyi.blackcat I/flutter: app main init
2021-10-09 16:45:03.620 1947-2171/com.xunyi.blackcat I/flutter: A-didPush route: /ARouteObserver

----------------------------------------------------------------------------------------------------------------

A页 -> B页 
Navigator.of(context).pushNamed('/BRouteObserver')


2021-10-09 16:45:38.684 1947-2171/com.xunyi.blackcat I/flutter: A-didPushNext route: /ARouteObserver
2021-10-09 16:45:38.715 1947-2171/com.xunyi.blackcat I/flutter: B-didPush route: /BRouteObserver

B页 -> C页
Navigator.of(context).pushNamed('/CRouteObserver')


2021-10-09 16:54:01.940 5429-5566/com.xunyi.blackcat I/flutter: B-didPushNext route: /BRouteObserver
2021-10-09 16:54:01.956 5429-5566/com.xunyi.blackcat I/flutter: C-didPush route: /CRouteObserver

结论：入栈时，会执行当前页的didPushNext  
----------------------------------------------------------------------------------------------------------------

C页 退回到 B页
Navigator.of(context).pop()


2021-10-09 16:54:20.297 5429-5566/com.xunyi.blackcat I/flutter: B-didPopNext route: /BRouteObserver
2021-10-09 16:54:20.297 5429-5566/com.xunyi.blackcat I/flutter: C-didPop route: /CRouteObserver


B页 退回到 A页
Navigator.of(context).pop()


2021-10-09 16:54:36.729 5429-5566/com.xunyi.blackcat I/flutter: A-didPopNext route: /ARouteObserver
2021-10-09 16:54:36.729 5429-5566/com.xunyi.blackcat I/flutter: B-didPop route: /BRouteObserver


结论： 返回到哪一页，哪一页先执行didPopNext 之后当前页再执行didPop动作

----------------------------------------------------------------------------------------------------------------

C页 替换成 A页 
Navigator.of(context).pushReplacementNamed('/ARouteObserver')


2021-10-09 17:20:39.036 12183-12312/com.xunyi.blackcat I/flutter: oldRoute : /CRouteObserver
2021-10-09 17:20:39.036 12183-12312/com.xunyi.blackcat I/flutter: newRoute : /ARouteObserver
2021-10-09 17:20:39.058 12183-12312/com.xunyi.blackcat I/flutter: A-didPush route: /ARouteObserver



替换为A页之后 退回上一页（B页）
Navigator.of(context).pop()


2021-10-09 17:26:27.502 12183-12312/com.xunyi.blackcat I/flutter: B-didPopNext route: /BRouteObserver
2021-10-09 17:26:27.503 12183-12312/com.xunyi.blackcat I/flutter: A-didPop route: /ARouteObserver

----------------------------------------------------------------------------------------------------------------

// 获取当前页的route名称
ModalRoute.of(context).settings.name
```



