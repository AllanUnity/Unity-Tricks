# Unity Tricks小把戏

# NO.1 Sprite切图
### 前期准备
> * 把图片`Texture Type`改为`Sprite`，`Sprite Mode`改为`Multiple`。
> * 打开`Sprite Editor`，左上角选择`Slice`，可以自动切图，或者手动调整，然后`Apply`，会自动生成一张张图片，但此时图片的信息只是存在meta中，需要导出图片。

### 测试
> * 脚本路径在Editor下的`PackerEditor`，属于UnityEditor类的，意味着不是在运行时工作，而是在编辑时工作。
> * Unity引擎会自动检测到Editor文件下的UnityEditor类，可以看到导航栏的Assets下多了个`Packer`。
> * 把图片`Texture Type`改为`Advanced`，将`Read/Write Enabled`属性进行打勾。
> * 选中切图后的图片，然后在菜单栏选择`Assets->Packer`，此时生成的图片放在同一路径上的同名文件夹。

# NO.2 Shader透视
### 原理
> * 将玩家绘制多遍：`第一遍透视绘制`，当玩家被物体挡住，则绘制绿色像素。`第二遍正常绘制`：当玩家没有被物体挡住，则绘制正常颜色。
> * Z测试和Z比较：`物体到镜头的距离`就是Z轴（深度值），镜头处为0，镜头最大截面为1，越小越靠近镜头。
> * Z开启和Z关闭：`ON`保留镜头的Z值，屏幕上每个像素点都存储有当前最靠近镜头的物体的Z值，`OFF`避免屏幕存储错误的Z值，因为玩家被物体挡住。如果`打开Z测试`（深度缓存），我们就可以根据此深度值决定同样映射到此处的像素值是否可以写入到帧缓存中并绘制到屏幕上（最后产生的效果就是离相机越近的像素只会被写入到帧缓存，并绘制出来）。如果`关闭Z测试`（深度缓存），那么像素是否写入帧缓存就不依赖距离相机的远近（依赖于程序代码实现）。
> * Shader其他小知识：  
`pos`：屏幕上每个像素的位置。  
`UV`：屏幕上每个像素的纹理坐标，用于从贴图里面读取像素。  
`顶点着色器`（Vertex）：它执行在场景中的每个点，输出坐标投影，颜色，纹理和其他数据传递到片段着色器。  
`片段着色器`（Fragment）：它执行在每个像素，输出信息为像素的颜色。  

### 前期准备
> * 场景包含2个Plane，下面的Plane作为地板，上面的Plane取消了`Mesh Collider`，可以挡着玩家，方便测试下透视效果，另外还有4个Cube作为围墙。
> * 玩家由Capsule和Cube组成，Shader文件夹下有一个`Material`用来控制玩家的材质。
> * 控制玩家移动的脚本，放在Scripts下的`PlayerMove`。
> * 把脚本挂到玩家身上，记得还要给玩家添加一个`Rigidbody`组件，并勾上`Constraints`->`Freeze Rotation`->`X Z`，这是控制玩家只能朝着Y轴方向。

### 测试
> * Shader文件夹下有一个`PlayerShader`，控制玩家透视效果。
> * Unity引擎会自动检测Shader文件，然后自动添加到Shader下拉列表。
> * 选择玩家`Shader`->`Custom`->`PlayerShader`。
> * 运行游戏，可以发现当玩家走到被物体挡住的地方时，会有黑色的阴影效果，成功。

# NO.3 文字打字效果
> * 场景中有`Text`，上面挂着`Typing`脚本。
> * 运行Main场景，可以看到文字呈现打字效果。

# NO.4 镜头震动效果
> * 场景中有`Camera`，上面挂着`Shaking`脚本。
> * 运行Main场景，可以看到镜头抖动效果。

# NO.5 玩家范围检测
### 原理
> * 玩家范围检测就是判断怪物是否在玩家一定`距离`和`角度`（呈现扇形区域）内，因此需要分别计算距离和角度。
> * 玩家和怪物的`距离`可以通过`Vector3.Distance`计算。
> * 玩家和怪物的`角度`可以通过`Mathf.Acos`计算，然后乘以`Mathf.Rad2Deg`转化为度数。

### 脚本解释
> * `Vector3.Distance(Vector3 a, Vector3 b)` 返回a和b之间的距离。
> * `Vector3.normalized` 向量标准化。（只读）
> * `Vector3.Dot(Vector3 lhs, Vector3 rhs)` 返回两个向量的点乘积。
> * `Mathf.Rad2Deg` 弧度到度的转化常量。（只读）
> * `Mathf.Acos(float f)` 以弧度为单位计算并返回参数 f 中指定的数字的反余弦值。
> * `Debug.DrawLine(Vector3 start, Vector3 end, Color color)` 从start起点到end末点，绘制一条color颜色的线。

### 测试
> * 场景有玩家和怪物，玩家设置同NO.2。
> * 范围检测脚本名为`Detecting`，挂到玩家身上。
> * 运行游戏，Game视图下点击Gizmos显示画图效果，可以看到如果范围内，则输出在，如果范围外，则输出不在。

# NO.6 镜头放大旋转
### 原理
> * `Vector.magnitude` 返回向量的长度，向量的长度是(x\*x+y\*y+z\*z)的平方根。
> * `Mathf.Clamp(float value, float min, float max)` 限制value的值在min和max之间，如果value小于min，返回min。如果value大于max，返回max。否则返回value。
> * `Transform.RotateAround(Vector3 point, Vector3 axis, float angle)` 一个物体围绕 point位置 的 axis轴 旋转 angle角度

### 脚本解释
> * transform.position = offsetPosition + player.position;  // 镜头跟随玩家
> * ScrollView();  // 控制镜头的拉近拉远  
`Input.GetAxis("Mouse ScrollWheel")` 鼠标向后滑动返回负数（拉近视野），向前滑动正数（拉远视野）
> * RotateView();  // 控制镜头的左右上下   
`Input.GetMouseButton(1)` 得到鼠标右键的按下  
`Input.GetAxis("Mouse X")` 得到鼠标水平方向的滑动  
`Input.GetAxis("Mouse Y")` 得到鼠标垂直方向的滑动  

### 测试
> * 场景有两个玩家（默认Player2可用），Player1设置同NO.2是用键盘WSAD控制玩家移动，Player2我们新增了一个脚本`PlayerMove2`来用鼠标左键控制玩家移动。
> * 控制镜头的脚本名为`FollowPlayer`，挂到镜头身上。
> * 运行游戏，鼠标`左键`可以控制玩家移动，鼠标`滑轮`可以控制镜头拉近拉远，鼠标`右键`可以控制镜头上下左右。

# NO.7 跑马灯
### 原理
> * 跑马灯有区域限制，超出这个区域就不显示，这里我们用`Mask遮罩`实现。
> * 以水平跑马灯为例：跑马灯的可视范围是背景宽度，文字从右边开始到左边结束，总共移动的距离是`背景宽度 + 文字宽度`。
> * 跑马灯的动画实现使用了`DOTween插件`。

### 前期准备
> * 新建一个Image作为背景。调整适当大小。
> * 背景下再新建一个Image。添加Mask组件，用于遮住背景之外的文字，Rect Transfrom设置为Stretch，四维全部设置为0，铺满背景。
如果是水平滚动的将Rect Transform的Pivot设置为`1 0.5`，令Mask锚点位于`右边`。
如果是垂直滚动的将Rect Transform的Pivot设置为`0.5 0`，令Mask锚点位于`下边`。
> * Mask下创建Text，随意写些文字，居中显示，添加Content Size Fitter。
如果是水平滚动的将`Horizontal Fit`设置为Preferred Size，将Rect Transform的Pivot设置为`0 0.5`，令Text锚点位于Mask处，方便实现从右往左动画。
如果是垂直滚动的将`Vertical Fit`设置为Preferred Size，将Rect Transform的Pivot设置为`0.5 1`，令Text锚点位于Mask处，方便实现从下往上动画。

### 脚本解释
```
float width = HText.preferredWidth;  // 获取文字的长度
HText.rectTransform.anchoredPosition = new Vector2(0, 0.5f);  // 让文字从在最右边开始移动
Tweener tweener = HText.rectTransform.DOLocalMoveX(-(bgWidth + width), duration);  // 设置动画持续时间
tweener.SetDelay(delay);  // 设置动画延迟时间
tweener.SetEase(Ease.Linear);  // 设置动画播放方式
tweener.SetLoops(5, LoopType.Restart);  // 每次播放结束后重新开始播放，一共播放 5 次
tweener.OnStart(delegate { Debug.Log("水平走马灯事件开始"); });  // 设置动画开始事件
tweener.OnComplete(delegate { Debug.Log("水平走马灯事件结束"); });  // 设置动画结束事件
```

# NO.8 淡入淡出效果
### 原理
> * 通过改变组件color的alpha值，实现隐藏。
> * 淡入淡出的动画实现使用了`DOTween插件`。

### 脚本解释
```
// delay时间内隐藏
Tweener tweener = GetComponent<Text>().DOFade(0, delay);
// 线性速度渐变
tweener.SetEase(Ease.Linear);
// -1表示无限循环
tweener.SetLoops(-1, LoopType.Yoyo);
```

# NO.9 摇杆功能
> * `定义委托`  
public delegate void JoyStickTouchBegin(Vector2 vec);  // 定义触摸开始事件委托  
public delegate void JoyStickTouchMove(Vector2 vec);  // 定义触摸过程事件委托  
public delegate void JoyStickTouchEnd();  // 定义触摸结束事件委托  
> * `注册事件`  
public event JoyStickTouchBegin OnJoyStickTouchBegin;  // 注册触摸开始事件  
public event JoyStickTouchMove OnJoyStickTouchMove;  // 注册触摸过程事件  
public event JoyStickTouchEnd OnJoyStickTouchEnd;  // 注册触摸结束事件  
> * `使用接口`  
IPointerDownHandler, IPointerUpHandler, IDragHandler  
public void OnPointerDown(PointerEventData eventData)  // 触摸开始  
public void OnPointerUp(PointerEventData eventData)  // 触摸结束  
public void OnDrag(PointerEventData eventData)  // 触摸过程  
> * `返回摇杆的偏移量`  
```
private Vector2 GetJoyStickAxis(PointerEventData eventData)
{
    // 获取手指位置的世界坐标
    Vector3 worldPosition;
    if (RectTransformUtility.ScreenPointToWorldPointInRectangle(selfTransform,
             eventData.position, eventData.pressEventCamera, out worldPosition))
        selfTransform.position = worldPosition;
    // 获取摇杆偏移量
    Vector2 touchAxis = selfTransform.anchoredPosition - originPosition;
    // 摇杆偏移量限制
    if (touchAxis.magnitude >= JoyStickRadius)
    {
        touchAxis = touchAxis.normalized * JoyStickRadius;
        selfTransform.anchoredPosition = touchAxis;
    }
    return touchAxis;
}
```

# NO.10 小地图功能
> * `UI准备`：Mask圆形遮罩，Minimap小地图边框。
> * 添加一个新的相机，并命名为`Mini Camera`。然后将该相机设为 Player 的子对象，position设为(0, 10,0)，rotation设为(90, 0, 0)。
> * 渲染到UI层需要用到Render Texture来实现。依次点击菜单项Assets -> Create -> Render Texture新建Render Texture，并命名为`Minimap Render`。选中Mini Camera后将Target Texture设为Minimap Render。
> * 下面新建Canvas来添加UI元素。新建Raw Image，命名为`Map`，将Texture设为Minimap Render。
> * 下面新建Image，命名为`Mask`，为其添加Mask组件，并将Image的Source Image设为上面的圆形遮罩。最后将Map设为Mask的子对象。
> * 下面新建Image，命名为`Outline`，将Image的Source Image设为上面的小地图边框。
> * 为了让整个小地图移动起来更方便，新建一个空的GameObject命名为`Minimap`，并将所有对象设为Minimap子对象。
> * 最后层级如下：  
`Minimap`  
---- `Mask`  
-------- `Map`  
---- `Outline`  

# NO.11 时间倒流效果
> * 一个简单的思路就是用Stack来记录物体的Position和Rotation，当需要时间回退的时候就Pop出来，赋值到物体上。不过为了可以进行拓展，比如只能回退到某段时间内的，而不是一下子回退到最开始的地方，我们需要剔除太久之前的信息。因此选择使用List而不是Stack。
```
// 限制实现
if (ShouldLimit && HistoryPos.Count > Limit)  // 是否限制列表长度
{
    HistoryPos.RemoveAt(0);
    HistoryRot.RemoveAt(0);
}
```
```
// 倒流实现
if (HistoryPos.Count > 0)
{
    int index = HistoryPos.Count - 1;
    this.transform.position = HistoryPos[index];
    HistoryPos.RemoveAt(index);
}
if (HistoryRot.Count > 0)
{
    int index = HistoryRot.Count - 1;
    this.transform.rotation = HistoryRot[index];
    HistoryRot.RemoveAt(index);
}
```

# NO.12 爆炸效果
> * 场景中有四个方块Cube作为受到爆炸影响的物体，给它们添加刚体组件。新建一个球体Sphere，作为炸弹。当炸弹落下时碰撞到地面，炸弹爆炸。
```
// 如果碰撞物是地面，进行爆炸处理
if (col.transform.tag == groundTag)
{
    // 定义爆炸位置为炸弹位置
    Vector3 explosionPos = transform.position;
    // 这个方法用来返回球型半径之内的所有碰撞体collider
    Collider[] colliders = Physics.OverlapSphere(explosionPos, radius);
    // 遍历返回的碰撞体，如果是刚体，则给刚体添加力
    foreach (Collider hit in colliders)
    {
        if (hit.GetComponent<Rigidbody>())
        {
            hit.GetComponent<Rigidbody>().AddExplosionForce(force, explosionPos, radius, ups);
        }
        // 销毁地面和炸弹
        Destroy(col.gameObject);
        Destroy(gameObject);
    }
}
```

# No.13 射线拾取效果
> * 场景中有四个方块Cube，当我们鼠标点击物品时，物品会消失。
```
// 检测鼠标左键的按下
if (Input.GetMouseButtonDown(0))
{
    // 创建一条射线，产生的射线是在世界空间中，从相机的近裁剪面开始并穿过屏幕 position(x,y) 像素坐标
    Ray ray = GetComponent<Camera>().ScreenPointToRay(Input.mousePosition);
    // RaycastHit 是一个结构体对象，用来储存射线返回的信息
    RaycastHit hit;
    // 如果射线碰撞到对象，把返回信息储存到 hit 中
    if (Physics.Raycast(ray, out hit))
    {
        // 销毁碰撞到的物品
        if (hit.transform.CompareTag(cubeTag))
        {
             Destroy(hit.transform.gameObject);
        }
    }
}
```

# NO.14 聊天框效果
> * 重点难点：  
1.需要控制别人和自己聊天框Item的位置  
2.需要控制聊天框ScrollView的滚动  
3.需要控制聊天框Item的宽度高度  
4.需要控制聊天框ScrollView的伸长  
5.需要移除历史聊天框Item  
> * 基本UI组件有玩家输入框、发送按钮、聊天框Item、聊天框ScrollView。
> * 聊天框Item有left和right两种，分别是别人和自己，以自己的聊天框right为例子:  
1.新建一个Image作为`背景`，设置Anchor为(right, top)、Pivot为(1, 1)。  
2.在背景下新建一个Image作为`头像`，设置Anchor为(right, bottom)和一个Text作为`文字`。  
3.在头像下新建一个Text作为`名字`，设置Anchor为(right, middle)。  
4.挂上ChatUI脚本，专门控制UI显示。
5.将其制作成为Prefab，聊天框left同理。  
> * 聊天框ScrollView：  
新建一个ScrollView，设置Anchor为(stretch, stretch)，调整为适当大小。  
```
// 重要参数
private float minWidth = 100.0f;  // 聊天框最小宽度
private float maxWidth = 400.0f;  // 聊天框最大宽度
private float iconHeight = 100.0f;  // 头像高度
private float chatHeight = 10.0f;  // 聊天框间隔
private float allHeight = 0.0f;  // 聊天框总高度
private float marginWidth = 40.0f;  // 宽度边距
private float marginHeight = 40.0f;  // 高度边距
private int historyCnt = 10;  // 历史条数
private List<GameObject> itemList = new List<GameObject>();  // 历史聊天框列表
```
```
// 控制适应聊天框
void FitScreen(GameObject tempGo)
{
    Text tempChatText = tempGo.transform.Find("Content").GetComponent<Text>();
    if (tempChatText.preferredWidth + marginWidth < minWidth)  // 单行宽度太短，宽度至少为 minWidth
    {
        tempGo.GetComponent<RectTransform>().sizeDelta = new Vector2(minWidth, tempChatText.preferredHeight + marginHeight);
        tempChatText.GetComponent<RectTransform>().sizeDelta = new Vector2(minWidth, tempChatText.preferredHeight + marginHeight);
    }
    else if (tempChatText.preferredWidth + marginWidth > maxWidth)  // 单行宽度太长，宽度至多为 maxWidth
    {
        tempGo.GetComponent<RectTransform>().sizeDelta = new Vector2(maxWidth, tempChatText.preferredHeight + marginHeight);
        tempChatText.GetComponent<RectTransform>().sizeDelta = new Vector2(maxWidth - marginWidth, tempChatText.preferredHeight + marginHeight);
    }
    else  // 不长不短，文字自适应聊天框
    {
        tempGo.GetComponent<RectTransform>().sizeDelta = new Vector2(tempChatText.preferredWidth + marginWidth, tempChatText.preferredHeight + marginHeight);
        tempChatText.GetComponent<RectTransform>().sizeDelta = new Vector2(tempChatText.preferredWidth, tempChatText.preferredHeight + marginHeight);
    }
    tempChatText.SetVerticesDirty();  // 通知 Layout 布局需要重建
    tempGo.GetComponent<RectTransform>().anchoredPosition = new Vector3(0f, -allHeight);  // 相对于中心点设置位置
    allHeight += (tempChatText.preferredHeight + marginHeight) + iconHeight + chatHeight;  // 增加高度，包括文字背景、头像高度和聊天框间隔
    if (allHeight > itemParent.GetComponent<RectTransform>().sizeDelta.y)  // 超出父容器，父容器伸长
    {
        itemParent.GetComponent<RectTransform>().sizeDelta = new Vector2(itemParent.GetComponent<RectTransform>().sizeDelta.x, allHeight);
    }
}
```

# NO.15 幸运转盘
> * 灯泡闪烁，不停地切换两张图片。
```
// 视觉上产生动态效果
InvokeRepeating("SwitchRound", 0.0f, switchRoundTime);
```
> * 获奖物品，根据角度计算决定奖品。
```
// 360 为旋转一圈度数，22.5f 为初始偏移度数，取余 360 防止下标越界
float angle = (360 - pointTransform.eulerAngles.z + 22.5f) % 360;
// 每个奖品区域 45 度
int index = (int)angle / 45;
/ 根据角度计算决定奖品
nameText.text = switchNames[index];
```
> * 指针顺时针旋转，指针旋转速度缓慢
```
// 指针顺时针旋转
pointTransform.Rotate(Vector3.back * rotateSpeed * Time.deltaTime);
// 指针旋转速度缓慢
rotateSpeed -= deltaSpeed;
```

# NO.16 卡牌轮换
> * 定义偏移量
```
private Vector3 Center = Vector3.zero;
private float offsetX = 0.6f;
private float offsetY = 0.1f;
private float offsetZ = 0.1f;
```
> * 保证卡牌数量为奇数，中间为标志位，两边刚好对称
```
// 初始化精灵数组
int childCount = transform.childCount;
// 计算两侧精灵数目
halfSize = (childCount - 1) / 2;
// 初始化精灵
sprites = new GameObject[childCount];
for (int i = 0; i < childCount; i++)
{
    sprites[i] = transform.GetChild(i).gameObject;
    SetPosition(i);
     // SetDeepin(i);
}
index = halfSize + 1;
// 计算精灵坐标
void SetPosition(int index)
{
    float x;
    float y;
    float z;
    if (index < halfSize)
    {
        x = -(halfSize - index) * offsetX + Center.x;
        y = (halfSize - index) * offsetY + Center.y;
        z = (halfSize - index) * offsetZ + Center.z;
    }
    else if (index > halfSize)
    {
        x = (index - halfSize) * offsetX + Center.x;
        y = (index - halfSize) * offsetY + Center.y;
        z = (index - halfSize) * offsetZ + Center.z;
    }
    else 
    {
        x = Center.x;
        y = Center.y;
        z = Center.z;
    }
    sprites[index].GetComponent<Transform>().position = new Vector3(x, y, z);
}
```

# NO.17 分页效果
### 前期准备
> * 制作Grid  
1.新建Image，改名`Grid`作为头像。  
2.新建Image作为`Grid`子物体，改名为`Item`作为物品名字背景。  
3.新建Text作为`Item`子物体，改名为`Name`作为物品名字。  
4.将物体制作成Prefab，最后层次关系应该是：
Grid  
----Item  
--------Name  
> * 自动排版  
1.新建Panel，将Grid作为Panel子物体，再将Grid复制12份。  
2.在Panel下添加`Grid Layout Group`组件，调整Padding、Cell Size、Spacing到合适位置，可以看到子物体全部自动排版。  

### 脚本解释
> * 重要属性
```
private int itemsCount = 0;  // 物品数量
private int pagesCount = 0;  // 页面数量
private int pageIndex = 1;  // 当前页面
private const int COUNT = 12;  // 一页物品数量
private Vector3 from = new Vector3(1f, 1f, 1f);  // 动态变大
private Vector3 to = new Vector3(0.8f, 0.8f, 0.8f);  // 动态变小
private List<GridItem> itemList = new List<GridItem>();  // 物品列表
```
> * 初始化列表
```
// 利用12生肖数组来随机生成列表
int cnt = Random.Range(1, 100);
for (int i = 0; i < cnt; ++i)
{
    itemList.Add(items[Random.Range(0, items.Length)]);
    Debug.Log(itemList[i].itemName);
}
// 计算元素总个数
itemsCount = itemList.Count;
// 计算总页数
pagesCount = (itemsCount % COUNT) == 0 ? itemsCount / COUNT : (itemsCount / COUNT) + 1;
BindPage(pageIndex);
// 更新界面页数
panelText.text = string.Format("{0} / {1}", pageIndex.ToString(), pagesCount.ToString());
```
> * 分页处理
```
// 需要特别处理的是最后1页
if (index == pagesCount)
{
    // 最后一页剩下的元素数目为 itemsCount - COUNT * (index - 1)
    // 其中 COUNT * (index-1) 为前面元素数目
    int cnt = itemsCount - COUNT * (index - 1);
    for (int i = 0; i < cnt; ++i)
    {
        BindGridItem(transform.GetChild(i), itemList[COUNT * (index - 1) + i]);
        transform.GetChild(i).gameObject.SetActive(true);
    }
    for (int i = cnt; i < COUNT; ++i)
    {
        // 隐藏多余物品
        transform.GetChild(i).gameObject.SetActive(false);
        }
    }
    // 其他情况正常显示
    else
    {
        for (int i = 0; i < COUNT; ++i)
        {
            BindGridItem(transform.GetChild(i), itemList[COUNT * (index - 1) + i]);
            transform.GetChild(i).gameObject.SetActive(true);
        }
    }
}
```

# NO.18 AssetBundle使用
### 前期准备
> * 导入角色文件夹，将角色制作成Prefab。
> * 点击Prefab，可以看到Inspector视图下方的AssetBundle。
> * 在Prefab的AssetBundle下添加一个名叫`swordman`的标识。

### 导出AssetBundle
> * 在同目录下新建目录`Editor`，进入目录新建脚本`BundleEditor`：
```
using UnityEditor;
using UnityEngine;
public static class BundleEditor
{
    private static string resPath = Application.dataPath + "/NO18/AssetBundles/";
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAssetBundles()
    {
        BuildPipeline.BuildAssetBundles(resPath, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows);
    }
}
```
> * 这里需要保证目录`AssetBundles`存在，否则导致报错。
> * 在菜单栏选择`Assets->Build AssetBundles`，AssetBundle将自动生成到对应目录。

### 测试AssetBundle
> * 从本地加载的文件目录：`"file://F:/HelloWorld/Unity/Unity Tricks/Assets/NO18/AssetBundles/"`。注意这里的地址就是我们生成的对应目录。
> * 从网络加载的文件目录：`"http://www.littleredhat1997.com/games/AssetBundles/"`。注意需要将生成的AssetBundle放到服务器对应目录。
> * 需要加载的资源名字：`"swordman"`
```
// 从本地 / 网络加载
IEnumerator LoadAssetBundles(string url)
{
    WWW www = new WWW(url);
    yield return www;
    if (www.error != null)
    {
        Debug.LogError("网络错误");
    }
    else
    {
        AssetBundle bundle = www.assetBundle;
        // 加载资源
        Object obj = bundle.LoadAsset(prefabName);
        go = Instantiate(obj) as GameObject;
        // 释放加载的资源
        bundle.Unload(false);
    }
}
```

# NO.19 录音功能
> * `开始录音`
Microphone.Start(string deviceName, bool loop, int lengthSec, int frequency)
> * `停止录音`
Microphone.End(string deviceName)
> * `播放录音`
audioSource.PlayOneShot(clip)
> * `保存录音`
Wav文件分好几个种类，相应的非数据信息存储在文件头部分，以下是其中一种WAV文件头格式。  
8KHz采样、16比特量化的线性PCM语音信号的WAVE文件头格式表（`共44字节`）  
偏移地址 字节数 数据类型 内容 文件头定义为  
00H 4 char "RIFF" char riff_id[4]="RIFF"  
04H 4 long int 文件总长-8 long int size0=文总长-8  
08H 8 char "WAVEfmt " char wave_fmt[8]  
10H 4 long int 10 00 00 00H(PCM) long int size1=0x10  
14H 2 int 01 00H int fmttag=0x01  
16H 2 int int channel=1 或2  
18H 4 long int 采样率 long int samplespersec  
1CH 4 long int 每秒播放字节数 long int bytepersec  
20H 2 int 采样一次占字节数 int blockalign=声道数*量化数/8  
22H 2 int 量化数 int bitpersamples=8或16  
24H 4 char "data" char data_id="data"  
28H 4 long int 采样数据字节数 long int size2=文长-44  
2CH 到文尾 char 采样数据  

# NO.20 2D攻击效果
> * Inspector面板特殊行为，使用`[Header("***")]`定义变量，可以将***内容显示在Inspector面板上。
> * 为了实现攻击特效，使用DOTween中的`DOPunchAnchorPos`实现玩家来回攻击以及`DOFade`实现血条淡入淡出。
> * 玩家攻击方向playDir分为左Vector2.Left和右Vector2.Right，需要在Inspector面板手动设置，其中玩家一(X 1, Y 0)，玩家二(X -1, Y 0)。
> * 玩家属性  
[Header("冲击力量")]  
public const int punch = 100;  
[Header("血量上限")]  
public const int allHp = 100;  
[Header("当前血量")]  
public int nowHp = 100;
```
// Back and Forth ==> punch duration
transform.GetComponent<RectTransform>().DOPunchAnchorPos(playDir * punch, 0.2f);
// Fade In ==> endValue duration
damageText.DOFade(1.0f, 0f);
// Fade Out ==> endValue duration
damageText.DOFade(0f, 1.5f);
```

# NO.21 日夜交替
> * 每小时移动角度 = 1 / 24 * 360度 = 15度
> * 初始偏移角度 = 6 * 15度 = 90度
> * 6:00 ==> Ligth X轴 0度
> * 12:00 ==> Ligth X轴 90度
> * 18:00 ==> Ligth X轴 180度
```
// 计算当前时间
now = (now + Time.deltaTime) % day;
// 计算光照角度
float degree = now * perDegree - offsetDegree;
// 改变Light角度
transform.rotation = Quaternion.Euler(degree, 0, 0);
```

# NO.22 场景读条加载
> * 导入资源，将Texture Type更改为`Sprite Mode`，另外可以根据需要在`Sprite Editor`进行九宫切图。
> * 如果切换场景出现光照异常，莫慌，点击Window下的`Lighting`，取消Auto，点击Build，此时Unity自动将烘焙场景并生成场景同名文件夹，下次切换场景就不会光照异常。
> * 为了防止场景加载过快，我们可以通过设置`allowSceneActivation`控制加载完毕后是否加入下一个场景。
> * 在协程中异步加载场景，progress的取值范围在0.1 - 1之间，但是`不会等于1`。即progress可能在0.9的时候就会直接进入新场景，所以需要分别控制两种进度`0.1 - 0.9`和`0.9 - 1`。
```
op.allowSceneActivation = false;  // 控制异步加载的场景暂时不进入
op.allowSceneActivation = true;  // 激活场景
```
```
while (op.progress < 0.9f)
{
    toProgress = startProgress + (int)(op.progress * 100);
    while (displayProgress < toProgress)
    {
        ++displayProgress;
        SetProgress(displayProgress);
        yield return null;
    }
    yield return null;
}
```
```
toProgress = 100;
while (displayProgress < toProgress)
{
    ++displayProgress;
    SetProgress(displayProgress);
    yield return null;
}
```

# NO.23 单摆效果
> * 创建一个`Sphere`球体作为重物，Scale修改为(0.2, 0.2, 0.2)，加上Rigidbody组件。
> * 创建一个`Capsule`胶囊体作为绳子，Position修改为(0, 1, 0)，Scale修改为(0.005, 1, 0.005)，加上Rigidbody组件，勾选Freeze PositionXYZ和Freeze RotationXY。
> * 最关键的一步到了，向绳子加上`Fixed Joint`固定关节组件，并将Sphere拖到Connected Body上。固定关节基于另一个物体来限制一个物体的运动。
> * 接下来需要控制绳子的质心以及重物的初始受力，分别在Rope和Sphere两个脚本实现
```
/*
    设置质心为绳子中心
    相对于变换的位置和旋转
*/
GetComponent<Rigidbody>().centerOfMass = centerOfMass;
```
```
/*
    添加一个力到刚体
    Impulse为冲力
*/
GetComponent<Rigidbody>().AddForce(Vector3.right * force, ForceMode.Impulse);
```

# NO.24 插值移动效果
### 原理
> * 线性插值 `Vector3.Lerp(Vector3 from, Vector3 to, float smoothing)` 。
> * 公式 `t = from + (to - from) * smoothing`。
> * from为初始位置，to为结束位置，smoothing为平滑速度，返回t为线性插值计算出来的向量，范围在 [0...1]之间。

### 前期准备
> * 创建2D精灵Sprite，命名为`Player`，作为我们的主角。
> * 通过前面的教程切图，制作人物休息和运动动画，按住CTRL选中几张帧动画图片拖到Inspector上的主角，可以快速生成动画，命名为`PlayerIdle`和`PlayerRun`，同时生成主角同名动画状态机Player。
> * 双击Player动画状态机可以直接打开Animator视图，将PlayerIdle和PlayerRun拖到视图，分别右键`Make Transition`。
> * 在Animator视图的左侧可以选择Parameters，创建Bool型参数`Run`，作为我们的转换条件。
> * 通过上面的步骤，我们设置PlayerIdle到PlayerRun的转换条件为Run `True`，PlayerRun到PlayerIdle的转化条件为Run `False`。
> * 这里有个细节需要注意，将两个转换上的`Has Exit Time`选项取消勾选，否则动画切换的时候会不及时，因为转换到下一个动画之前必须等待当前动画播放完毕。

### 脚本解释
> * 控制人物朝向
```
// 向左移动
if (h < 0)
{
    // 人物Y轴翻转
    transform.eulerAngles = new Vector3(0, 180.0f, 0);
}
// 向右移动
else if (h > 0)
{
    // 人物Y轴翻转 
    transform.eulerAngles = Vector3.zero;
}
```
> * 控制人物线性插值移动和播放动画
```
if (Mathf.Abs(h) >= minDistance || Mathf.Abs(v) >= minDistance)
{
    // 获取移动方向
    Vector3 tarDir = new Vector3(h, v, 0);
    // 控制玩家移动
    // transform.Translate(tarDir * speed * Time.deltaTime);
    transform.position = Vector3.Lerp(transform.position,
        transform.position + tarDir * speed, smoothing * Time.deltaTime);
    if (!isRun)
    {
        isRun = true;
        // 控制玩家运动动画
        anim.SetBool("Run", true);
    }
}
else
{
    isRun = false;
    // 播放玩家休息动画
    anim.SetBool("Run", false);
}
```

# NO.25 弹力球效果
> * 创建一个Cube作为平面和一个Sphere作为小球，调整Position和Scale。
> * 添加Rigidbody组件到小球上，确保重力存在。
> * 创建一个Physic Material，命名为Bounce，将其Bounciness属性设置为1。
> * 将Bounce材质赋给Cube和Sphere，运行游戏即可看到弹力球效果。
> * 最后我发现了一个问题，小球到最后越弹越高，按理说应该是能量守恒的呀......

# NO.26 截屏功能
```
void Application.CaptureScreenshot(string fileName, int superSize);  
参数filename为截屏文件名称，superSize为放大系数，默认为0，即不放大。  
方法用于截取当前游戏画面并将截取的图片保存为PNG格式。截屏后文件会默认保存在根目录下，如果根目录下已存在同名文件，将会被替换。当superSize大于1时，截屏文件的宽度和高度将同时被放大superSize倍。
```

# NO.27 拍照功能
```
// 请求用户授权
AnyncOperation Application.RequestUserAuthorization(UserAuthorization.WebCam)
// 用户是否具有授权
bool Application.HasUserAuthorization(UserAuthorization.WebCam)
// 创建网络相机贴图
WebCamTexture(string deviceName, string requestedWidth, string requestedHeight, int requestedFPS)
// 播放
void WebCamTexture.Play()
// 创建二维纹理
Texture2D(int width, int height, TextureFormat format, bool mipmap);
// 读取屏幕像素信息并存储为纹理数据
void WebCamTexture.ReadPixels(Rect source, int destX, int destY, bool recalculateMipmaps)
// 应用
void WebCamTexture.Apply()
// 纹理编码为PNG格式
byte[] WebCamTexture.EncodeToPNG()
// 暂停
void WebCamTexture.Pause()
```

# NO.28 画图功能
### 前期准备
> * 创建一个空物体，添加LineRender组件，制作预制体。

### 脚本解释
```
// 鼠标左键按下瞬间
if (Input.GetMouseButtonDown(0))
{
    go = Instantiate(linePrefab, linePrefab.transform.position, transform.rotation);
    line = go.GetComponent<LineRenderer>();
    // 设置材质
    line.material = new Material(Shader.Find("Particles/Additive"));
    // 设置颜色
    line.startColor = Color.red;
    line.endColor = Color.red;
    // 设置宽度  
    line.startWidth = 0.1f;
    line.endWidth = 0.1f;
    i = 0;
}
// 鼠标左键按下期间
if (Input.GetMouseButton(0))
{
    i++;
    // 设置顶点数  
    line.numPositions = i;
    // 设置顶点位置 
    line.SetPosition(i - 1, Camera.main.ScreenToWorldPoint(
            new Vector3(Input.mousePosition.x, Input.mousePosition.y, 15)));
}
```

---
注：  
视图有宽屏1280\*720和长屏720\*1280两种，如果视图错误，请自行调整视图！！！  
部分代码和文字来自网络，经过本人整合到本工程，有任何不明白都可以与我交流！！！  