# flutter滚动组件系列



[toc]

## ListView

> 在flutter中，超出屏幕会发生溢出，通过使用Scrollable组件做到滚动效果

- `children`: 可数的子Widget依次排列的ListView

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211012151855108.png" alt="image-20211012151855108" style="zoom:30%;" />

```dart
ListView(
  children: [
    FlutterLogo(size: 400,),
    FlutterLogo(size: 400,),
  ],
),
```



- `builder`: 建造itemView的ListView

  - itemCount : itemView个数，不指定为无限

  - scrollDirection :  滚动方向，Axis.vertical 【垂直】和 Axis.horizontal 【横向】

  - itemExtent: 固定主轴方向的每个item高度（交叉轴为宽度，反之亦然）；

    - 若确定每个item尺寸，可以提高性能。方便滚动时的计算 【设置之后，itemBuilder的widget尺寸设置就不再有用】
    - **交叉轴方向，同ListView视窗尺寸**

  - itemBuilder: 构建itemView的函数，调用时会多加载一些用于缓存；

    - cacheExtent:  指定缓存大小，单位逻辑像素  （竖直方向）**缓存高度默认为1/3屏幕高度，而非缓存个数**

      ​						 设置为0，则不缓存

  - padding : 对于整个ListView而言的间距，在竖直方向，只有最上和最下才有间距，而不是中间就形成

  - scrollDirection :  滚动方向，Axis.vertical 【垂直】和 Axis.horizontal 【横向】

    

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211013142437737.png" alt="image-20211013142437737" style="zoom:30%;" />

```dart
ListView.builder(
  padding: const EdgeInsets.all(20.0),
  itemCount: 88,
  itemExtent: 50,
  scrollDirection: Axis.vertical,
  itemBuilder: (BuildContext context, int index) {
    return Container(
      color: index % 2 == 0 ? Colors.blue : Colors.blue[200],
      alignment: Alignment.center,
      child: Text('$index'),
    );
  },
)

I/flutter (26467): building 0
I/flutter (26467): building 1
I/flutter (26467): building 2
I/flutter (26467): building 3
I/flutter (26467): building 4
I/flutter (26467): building 5
I/flutter (26467): building 6
I/flutter (26467): building 7
I/flutter (26467): building 8
I/flutter (26467): building 9
I/flutter (26467): building 10
I/flutter (26467): building 11
I/flutter (26467): building 12
I/flutter (26467): building 13
I/flutter (26467): building 14
I/flutter (26467): building 15

```



- `separated`：带有分割线的ListView
  - separatorBuilder ： 构建分割线
  - 其他同`builder`

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211012155159501.png" alt="image-20211012155159501" style="zoom:30%;" />

```dart
ListView.separated(
  separatorBuilder: (BuildContext context, int index){
    if(index == 0){
      return Divider(thickness: 4, color: Colors.black54,);
    }
    return Divider(thickness: 1, color: Colors.red,);
  },
  itemCount: 88,
  itemBuilder: (BuildContext context, int index) {
    print('building $index');
    return Container(
      height: 50,
      alignment: Alignment.center,
      child: Text('$index'),
    );
  },
)
```

- `controller`:  滚动控制器

  - ```dart
    _scrollController.jumpTo(0.0) /// 生硬的直接跳转到顶部
    
    _scrollController.animateTo(0.0,
    	duration: Duration(seconds: 1),
    	curve: Curves.bounceOut,
    ); /// 动画方式跳转
    ```



- `physics` : 物理引擎
  - **ClampingScrollPhysics** -- Android默认的上拉下拉效果
  - **BouncingScrollPhysics** -- IOS默认的上拉下拉效果
  - **NeverScrollableScrollPhysics** -- 无法物理滚动
- `ListTile` : tile译为瓦片；定义在ListView中的规范化的itemView
  - 包含一些固定位置的widget属性，方便使用

```dart
Scaffold(
    appBar: AppBar(
      title: GestureDetector(
          onTap: () {
            _scrollController.animateTo(
              -20, // 为了弹回效果明显，使用负数
              duration: Duration(seconds: 1),
              curve: Curves.bounceOut,
            );
          },
          child: Text('ListView Demo')),
    ),
    body: Scrollbar(
      child: ListView.builder(
        controller: _scrollController,
        padding: const EdgeInsets.symmetric(vertical: 10, horizontal: 8),
        itemCount: 24,
        scrollDirection: Axis.vertical,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            tileColor: index % 2 == 0 ? Colors.orange : Colors.yellow,
            leading: Icon(Icons.person, color: Colors.black),
            title: Text('title_$index'),
            subtitle: Text('subtitle'),
            trailing: Icon(Icons.delete),
          );
        },
      ),
    )
)
```



### Scrollbar

> 包裹一层滚动条

- **CupertinoScrollbar** -- IOS风格的滚动条，支持拖拽滚动
- **isAlwaysShown** :  为true时，一直显示滚动条
  - 配合`controller`使用，并同ListView的controller使用同一个，产生联动效果



```dart
CupertinoScrollbar(
  isAlwaysShown: true,
  controller: _scrollController,
  child: ListView.builder(
    itemCount: 24,
    controller: _scrollController,
    itemBuilder: (BuildContext context, int index) {
      return ListTile(
        tileColor: index % 2 == 0 ? Colors.orange : Colors.yellow,
        leading: Icon(Icons.person, color: Colors.black),
        title: Text('title_$index'),
        subtitle: Text('subtitle'),
        trailing: Icon(Icons.delete),
      );
    },
  ),
)
```



### RefreshIndicator

> 下拉刷新



- onRefresh： 刷新函数 

  ```dart
  typedef RefreshCallback = Future<void> Function();
  ```

- backgroundColor : 刷新球背景色
- color : 刷新球颜色
- strokeWidth ：  刷新球粗细

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211013175954460.png" alt="image-20211013175954460" style="zoom:30%;" />

>  嵌套滚动

- 通过同一个**ScrollController**，联动滚动效果
- 通过**NotificationListener**监听滚动事件，来消费或处理滚动事件；返回true，则上层不再接收到滚动事件

```dart
CupertinoScrollbar(
  controller: _scrollController,
  child: RefreshIndicator(
    backgroundColor: Colors.black12,
    color: Colors.green,
    strokeWidth: 4.0,
    onRefresh: () {
      return Future.delayed(Duration(seconds: 5));
    },
    child: NotificationListener(
      onNotification: (ScrollNotification event){
        print(event);
        return false;
      },
      child: ListView.builder(
        itemCount: 24,
        controller: _scrollController,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            tileColor: index % 2 == 0 ? Colors.orange : Colors.yellow,
            leading: Icon(Icons.person, color: Colors.black),
            title: Text('title_$index'),
            subtitle: Text('subtitle'),
            trailing: Icon(Icons.delete),
          );
        },
      ),
    ),
  ),
)
  
I/flutter ( 2332): ScrollStartNotification(depth: 0 (local), FixedScrollMetrics(6.4..[683.4]..1062.2), DragStartDetails(Offset(255.2, 119.6)))
I/flutter ( 2332): UserScrollNotification(depth: 0 (local), FixedScrollMetrics(6.4..[683.4]..1062.2), direction: ScrollDirection.forward)
I/flutter ( 2332): ScrollUpdateNotification(depth: 0 (local), FixedScrollMetrics(4.8..[683.4]..1063.7), scrollDelta: -1.5401785714285694, DragUpdateDetails(Offset(0.0, 1.5)))
I/flutter ( 2332): ScrollUpdateNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), scrollDelta: -4.837218066535093, DragUpdateDetails(Offset(0.0, 10.6)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -5.8, velocity: 0.0, DragUpdateDetails(Offset(0.0, 10.6)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -2.7, velocity: 0.0, DragUpdateDetails(Offset(0.0, 2.7)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -5.3, velocity: 0.0, DragUpdateDetails(Offset(0.0, 5.3)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -6.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 6.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -6.5, velocity: 0.0, DragUpdateDetails(Offset(0.0, 6.5)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -8.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 8.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -6.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 6.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -6.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 6.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -8.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 8.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -16.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 16.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -9.5, velocity: 0.0, DragUpdateDetails(Offset(0.0, 9.5)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -10.7, velocity: 0.0, DragUpdateDetails(Offset(0.0, 10.7)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -13.3, velocity: 0.0, DragUpdateDetails(Offset(0.0, 13.3)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -13.3, velocity: 0.0, DragUpdateDetails(Offset(0.0, 13.3)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -16.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 16.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -16.4, velocity: 0.0, DragUpdateDetails(Offset(0.0, 16.4)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -16.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 16.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -16.0, velocity: 0.0, DragUpdateDetails(Offset(0.0, 16.0)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -22.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 22.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -14.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 14.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -11.8, velocity: 0.0, DragUpdateDetails(Offset(0.0, 11.8)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -13.3, velocity: 0.0, DragUpdateDetails(Offset(0.0, 13.3)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -12.2, velocity: 0.0, DragUpdateDetails(Offset(0.0, 12.2)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -10.7, velocity: 0.0, DragUpdateDetails(Offset(0.0, 10.7)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -12.2, velocity: 0.0, DragUpdateDetails(Offset(0.0, 12.2)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -10.7, velocity: 0.0, DragUpdateDetails(Offset(0.0, 10.7)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -9.5, velocity: 0.0, DragUpdateDetails(Offset(0.0, 9.5)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -9.1, velocity: 0.0, DragUpdateDetails(Offset(0.0, 9.1)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -4.2, velocity: 0.0, DragUpdateDetails(Offset(0.0, 4.2)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -6.9, velocity: 0.0, DragUpdateDetails(Offset(0.0, 6.9)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -3.8, velocity: 0.0, DragUpdateDetails(Offset(0.0, 3.8)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -4.2, velocity: 0.0, DragUpdateDetails(Offset(0.0, 4.2)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -2.7, velocity: 0.0, DragUpdateDetails(Offset(0.0, 2.7)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -1.1, velocity: 0.0, DragUpdateDetails(Offset(0.0, 1.1)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -1.5, velocity: 0.0, DragUpdateDetails(Offset(0.0, 1.5)))
I/flutter ( 2332): OverscrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), overscroll: -1.1, velocity: 0.0, DragUpdateDetails(Offset(0.0, 1.1)))
I/flutter ( 2332): UserScrollNotification(depth: 0 (local), FixedScrollMetrics(0.0..[683.4]..1068.6), direction: ScrollDirection.reverse)
I/flutter ( 2332): ScrollUpdateNotification(depth: 0 (local), FixedScrollMetrics(1.1..[683.4]..1067.4), scrollDelta: 1.1383928571428328, DragUpdateDetails(Offset(0.0, -1.1)))
I/flutter ( 2332): ScrollUpdateNotification(depth: 0 (local), FixedScrollMetrics(2.7..[683.4]..1065.9), scrollDelta: 1.5401785714285552, DragUpdateDetails(Offset(0.0, -1.5)))
I/flutter ( 2332): ScrollUpdateNotification(depth: 0 (local), FixedScrollMetrics(3.8..[683.4]..1064.8), scrollDelta: 1.1383928571428328, DragUpdateDetails(Offset(0.0, -1.1)))
I/flutter ( 2332): ScrollEndNotification(depth: 0 (local), FixedScrollMetrics(3.8..[683.4]..1064.8), DragEndDetails(Velocity(0.0, 0.0)))
I/flutter ( 2332): UserScrollNotification(depth: 0 (local), FixedScrollMetrics(3.8..[683.4]..1064.8), direction: ScrollDirection.idle)
  
  
```



### Dismissible

> 侧滑删除控件

- `key` :  区分widget的标识 【必填】

- `confirmDismiss` : 侧滑确认删除函数 

  - ```dart
    typedef ConfirmDismissCallback = Future<bool?> Function(DismissDirection direction);
    ```

  - 返回true, 才会回调`onDismissed` ,  同时支持同步操作，如提示弹窗之后再返回bool

- `onDismissed` ： 侧滑删除函数

  - ```dart
    typedef DismissDirectionCallback = void Function(DismissDirection direction);
    ```

  - 参数为 DismissDirection，回调后可以知道侧滑方向

    - DismissDirection.startToEnd /  DismissDirection.endToStart

- `background` :  **startToEnd**侧滑后的背景，接收一个widget 【没设置secondaryBackground 默认左右侧滑均可】

- `secondaryBackground` :  **endToStart**方向的侧滑后的背景，接收一个widget

- `resizeDuration` :  侧滑消失（尺寸变化）时长

- `onResize` :   侧滑消失（尺寸变化）回调

- `movementDuration` ： 拖动侧滑后放手后，侧滑的时长

- `dismissThresholds` :  拖动侧滑的长度阈值，才会视为开始**dismiss**

  - ```dart
    dismissThresholds: {
      DismissDirection.startToEnd: 0.1,
      DismissDirection.endToStart: 0.4,
    },
    ```

  

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211014105303649.png" alt="image-20211014105303649" style="zoom:30%;" /><img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211014105359743.png" alt="image-20211014105359743" style="zoom:30%;" />

```dart
Dismissible(
  key: UniqueKey(),
  background: Container(
    color: Colors.green,
    alignment: Alignment.centerLeft,
    padding: EdgeInsets.only(left: 16),
    child: Icon(Icons.phone),
  ),
  secondaryBackground: Container(
    color: Colors.blue,
    alignment: Alignment.centerRight,
    padding: EdgeInsets.only(right: 16),
    child: Icon(Icons.sms),
  ),
  resizeDuration: Duration(seconds: 3),
  onResize: (){
    print('onResize');
  },
  movementDuration: Duration(seconds: 13),
  dismissThresholds: {
    DismissDirection.startToEnd: 0.1,
    DismissDirection.endToStart: 0.4,
  },
  confirmDismiss: (direction) async {
    return direction == DismissDirection.endToStart;
  },
  onDismissed: (direction){
    print(direction);
  },
  child: Container(
    height: 72,
    color: index % 2 == 0 ? Colors.orange : Colors.yellow,
    child: ListTile(
      leading: Icon(Icons.person, color: Colors.black),
      title: Text('title_$index'),
      subtitle: Text('subtitle'),
      trailing: Icon(Icons.delete),
    ),
  ),
)
```

## 实例

> Github提供的API作为数据源

```dart
http: ^0.13.4


import 'package:http/http.dart' as http;
import 'dart:convert' as convert;
class _DemoListViewWidgetState extends State<DemoListViewWidget> {

  List<GitEvent> _events = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CupertinoScrollbar(
        child: RefreshIndicator(
          onRefresh: () async{
            await _refresh();
          },
          child: ListView(
            children: _events.map<Widget>((GitEvent event) {
              return Dismissible(
                key: ValueKey(event.id),
                confirmDismiss: (_) async {
                  return showDialog(
                      context: context,
                      builder: (_) {
                        return AlertDialog(
                          title: Text('Are you sure?'),
                          content: Text('Do you want to delete this item?'),
                          actions: [
                            TextButton(
                              child: Text('Cancel'),
                              onPressed: () {
                                  Navigator.of(context).pop(false);
                              },
                            ),
                            TextButton(
                              child: Text('Delete', style: TextStyle(color: Colors.red,),),
                              onPressed: () {
                                  Navigator.of(context).pop(true);
                              },
                            ),
                          ],
                        );
                      }
                  );
                },
                onDismissed: (_){
                  setState(() {
                    _events.removeWhere((element) => element.id == event.id);
                  });
                },
                child: ListTile(
                  leading: Image.network(event.avatarUrl ?? ''),
                  title: Text('${event.userName}'),
                  subtitle: Text('${event.repoName}'),
                ),
              );
            }).toList(),
          ),
        ),
      ),
    );
  }

  _refresh() async {
    final res =
    await http.get(Uri.parse('https://api.github.com/events'));
    if (res.statusCode == 200) {
      final json = convert.jsonDecode(res.body);
      setState(() {
        _events.clear();
        _events.addAll(json.map<GitEvent>((item) => GitEvent(item)));
      });
    }
  }
}

class GitEvent{
  String? id;
  String? userName;
  String? avatarUrl;
  String? repoName;

  GitEvent(json){
    this.id = json['id'];
    this.userName = json['actor']['login'];
    this.avatarUrl = json['actor']['avatar_url'];
    this.repoName = json['repo']['name'];
  }

  @override
  String toString() {
    return 'GitEvent{id: $id, userName: $userName, avatarUrl: $avatarUrl, repoName: $repoName}';
  }
}

```

## GridView

> 二维网格列表

- `builder`: 建造itemView的GridView
  - **gridDelegate** :  必填项 SliverGridDelegate
  - 其余同ListView.builder

### SliverGridDelegateWithFixedCrossAxisCount

> 通过确定**`交叉轴上容纳的固定个数`**，按1:1宽高比，算出网格

- crossAxisCount ： 交叉轴个数
- childAspectRatio :  itemView宽高比
- mainAxisSpacing :  主轴itemView间隙
- crossAxisSpacing：交叉轴itemView间隙

​	

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211014145146506.png" alt="image-20211014142836538" style="zoom:30%;" align="center" />



```dart
GridView.builder(
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 4,
      childAspectRatio: 16 / 9,
      crossAxisSpacing: 3,
      mainAxisSpacing: 3,
    ),
    itemCount: 100,
    itemBuilder: (_,index) {
      return Container(color: Colors.green[index % 8 * 100 + 100],);
    },
)

非动态可以使用： GridView.count(crossAxisCount: 4)
```



### SliverGridDelegateWithMaxCrossAxisExtent

> 通过确定**`交叉轴上最大限度`**，算出网格；在最大限度内，均可展示

- maxCrossAxisExtent ： 交叉轴上最大限度
- childAspectRatio ： itemView宽高比

```
GridView.builder(
    gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
      maxCrossAxisExtent: 100,
      childAspectRatio: 16 / 9,
    ),
    itemCount: 100,
    itemBuilder: (_,index) {
      return Container(color: Colors.red[index % 8 * 100],);
    },
),

非动态可以使用： GridView.extent(maxCrossAxisExtent: 4)
```



## Slivers

> sliver译为：一小片；就是关于片段itemView的一套小组件库



### CustomScrollView -- 视窗

> 进入Sliver的世界需要借助视窗作为桥梁，一般使用**CustomScrollView**

- `slivers` :  必填项，`Slivers相关的组件列表`



### Slivers组件

#### 一、SliverToBoxAdapter

> 将`RenderBoxWidget`【常用的widget】转化成`Sliver组件`的适配器

```
SliverToBoxAdapter(
  child: FlutterLogo(size: 100,),
)
```

#### 二、SliverList

> **ListView、GridView内部就是使用Sliver的委托类处理的**

- delegate ： 接收一个Sliver的委托类

  - **SliverChild`Builder`Delegate** --  动态加载建造itemView使用；ListView.builder等方式的底层实现

  - **SliverChild`List`Delegate** -- 固定个数itemView使用; 同直接使用ListView

  - ```dart
    ListView.builder({
      ...
    }) : 
         childrenDelegate = SliverChildBuilderDelegate(
           itemBuilder, // 列表widget
           childCount: itemCount, // 列表格式
           addAutomaticKeepAlives: addAutomaticKeepAlives,
           addRepaintBoundaries: addRepaintBoundaries,
           addSemanticIndexes: addSemanticIndexes,
         ),
    ```

#### 三、SliverFixedExtentList 【确定尺寸】

> 同ListView的 itemExtent，固定每项itemView主轴方向尺寸时使用

```dart
SliverFixedExtentList(
  itemExtent: 80,
  delegate: SliverChildBuilderDelegate((_, index) {
    return Container(
      color: Colors.primaries[index % Colors.primaries.length],
    );
  }),
),
```

#### 四、SliverPrototypeExtentList 【不确定尺寸】

> 同SliverFixedExtentList，但若无法确定item主轴方向尺寸，可以通过设置一个不可见的原型，计算完成后，会像SliverFixedExtentList一样使用该尺寸，设置itemView【推荐】

```dart
SliverPrototypeExtentList(
  prototypeItem: Container(
    height: 100,
  ),
  delegate: SliverChildBuilderDelegate((_, index) {
    return Container(
      color: Colors.primaries[index % Colors.primaries.length],
    );
  }),
)

// prototypeItem的widget不会真正显示，只是用来计算的，计算出的尺寸会像SliverFixedExtentList一样使用该尺寸，设置itemView
```



#### 四、SliverGrid

> GridView内部的Sliver方式

- delegate ： 同SliverList的delegate
- gridDelegate ： 同GridView的gridDelegate 【实际是一个东西】 

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211014160622291.png" alt="image-20211014160622291" style="zoom:50%;" />

```dart
CustomScrollView(
  physics: BouncingScrollPhysics(),
  slivers: [
    SliverToBoxAdapter(
      child: FlutterLogo(size: 100,),
    ),
    SliverGrid(
      delegate: SliverChildBuilderDelegate((_, index) {
        return Container(
          height: 200,
          color: Colors.primaries[index % Colors.primaries.length],
        );
      },childCount: 23,),
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 4,
        childAspectRatio: 16 / 9,
        mainAxisSpacing: 3,
        crossAxisSpacing: 3,
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate((_, index) {
        return Container(
          height: 100,
          color: Colors.primaries[index % Colors.primaries.length],
        );
      }),
    ),
  ],
)
```

#### 五、SliverFillViewport

> 每个itemView填充视窗

- viewportFraction比例默认为1，占据整个视窗；

<img src="/Users/shijiewang/Library/Application Support/typora-user-images/image-20211014163222467.png" alt="image-20211014163222467" style="zoom:30%;" />

```dart
SliverFillViewport(
		delegate: SliverChildBuilderDelegate((_, index) {
			return Container(
        color: Colors.primaries[index],
        child: Text(
          'Hello',
          style: TextStyle(fontSize: 36, color: Colors.yellow),
        ),
      );
		}, childCount: 6,),
   viewportFraction: 0.5, // 占据半个视窗
),
```