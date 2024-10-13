---
title: 学习笔记：Android Jetpack Compose 中的 Navigation
date: 2024-10-06 12:12:40
tags:
 - Android
 - Compose
 - Jetpack
categories:
 - Note
---

## 介绍 
Ref: 
[使用 Compose 实现多屏幕导航](https://developer.android.com/codelabs/basic-android-kotlin-compose-navigation?hl=zh-cn)
[使用 Compose 进行导航](https://developer.android.com/develop/ui/compose/navigation?hl=zh-cn)
[Jetpack Compose学习(11)——Navigation页面导航的使用](https://www.cnblogs.com/stars-one/p/17154864.html)

在 Jetpack Compose 中可以使用 Navigation 在不同屏幕之间导航，需要实现下面三个部分

**NavHost**：定义导航图，并指定起始目的地及其他可导航的目的地(NavGraph)。
**NavGraph**：用于映射要导航到的可组合项目标页面。  
**NavController**：负责在目标页面（即应用中的屏幕）之间导航，可在目的地之间导航、处理深层链接、管理返回堆栈等。  

<!--more-->

## 使用
### 添加依赖
```kotlin
implementation("androidx.navigation:navigation-compose:2.8.2")
```

or 
```ini
[versions]
nav = "2.8.2"  

[libraries]
navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "nav" }
```

```kotlin
implementation(libs.navigation.compose)
```

### 为应用添加 `NavHost`
NavHost 是一个可组合项，用于根据给定路线来显示其他可组合项目标页面，结构如下图 
![NavHost](https://m.nep.me/blog/post/compose-navigation-navhost.png)   

- `navController` 是 `NavHostController` 类实例，可以通过从组合函数调用 `var navControlled = rememberNavController()` 来获取。
- `startDestination` 此字符串路线用于定义应用首次显示 `NavHost` 时默认显示的目标页面。  
- 在 `content` 部分，可以调用 `composable()` 函数定义目的地路由

### 定义 composable 目的地
使用 `composable()` 函数为屏幕（目的地）定义路由，路由本质可以认为是一段字符串，类似于网址 

![route](https://m.nep.me/blog/post/compose-navigation-route.png)  

`composable()` 函数有两个必需参数。 
- `route`：与路线名称对应的字符串。这可以是任何唯一的字符串。
- `content`：您可以在此处调用要为特定路线显示的 `@Composable` 可组合项。

简单的示例：
```kotlin
import androidx.navigation.compose.composable

@Composable
fun NavigationApp(){
    val navController = rememberNavController()
    NavHost(
        navController,
        startDestination = "Home"
    ){
        composable("Home") {
            HomeScreen(navController)
        }
        composable("details/{itemId}") { backStackEntry ->
            val itemId = backStackEntry.arguments?.getString("itemId")
            DetailsScreen(itemId)
        }
    }
}
```

在不同的屏幕之间导航可以使用 `navController.navigation("route")` 
返回上一个屏幕，可以调用 `navController.navigateUp()`

```kotlin
composable("Home") {
    HomeScreen(navController)
}
```

表示导航到 **"Home"** 路由，显示 HomeScreen()，可以直接传递 navController 或是通过状态提升传递函数

```kotlin
composable("Home") {
    HomeScreen(
        onButtonClicked = { navController.navigation("Next") }
    )
}

// HomeScreen.kt
@Composable
fun HomeScreen(
    onButtonClicked: () -> Unit,
    modifier: Modifier = Modifier
) {
    //...
    Button( 
        onClick = onButtonClicked 
    ){ Text("Button") }
    //...
}
```

### 导航参数 
在 Navigation 组件 中，导航可以添加参数，允许在不同的屏幕（Composable）之间传递数据。适合于展示特定内容（如详情页面）。  
比如 
```kotlin
composable("details/{itemId}") { backStackEntry ->
    val itemId = backStackEntry.arguments?.getString("itemId")
    DetailsScreen(itemId)
}
```

路由路径 "details/{itemId}" 定义了一个带有参数的导航目的地：

`details`：这是路由的固定部分，表示这是一个详情页面。
`{itemId}`：花括号 {} 包围的部分表示一个动态参数，占位符，用于接收具体的 itemId 值，默认为 String 类型，也可以指定类型。

`DetailsScreen()` 应该设计为可以接收 `itemId`
```kotlin
@Composable
fun DetailsScreen(
    itemId : String
){
    //...
}
```

指定类型
```kotlin
composable("details/{id}", arguments = listOf(navArgument("id"){
    type = NavType.IntType
})){
    //...
}
```

navArgument 可以设置以下三个属性
- `type` 类型
- `defaultValue` 默认值
- `nullable` 是否可空

**可选参数**
上面的例子,实际上是必传的一个参数,如果需要可选参数,有以下规则:

- 可选参数必须使用查询参数语法 ·?argName={argName}· 来添加
- 可选参数必须具有 `defaultValue` 或 `nullable = true` (将默认值设置为 `null`)  
第二点就是上面提到过的两个属性,例如我们加多一个age的可选参数:

```kotlin
composable("details/{id}?age={age}", arguments = listOf(
    navArgument("id") {
        type = NavType.IntType
    }, navArgument("age") {
        type = NavType.IntType
        defaultValue = 0
    }
)) {
    //省略...
}
```

**简单示例**
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            NavigationArgsTheme {
                SimpleApp()
            }
        }
    }
}

enum class Screen{
    Home,
    Details
}

@Composable
fun SimpleApp(){
    val navController = rememberNavController()

    NavHost(
        navController,
        startDestination = Screen.Home.name
    ){
        composable(Screen.Home.name) {
            HomeScreen { user: String ->
                val username = user.ifBlank { "default" }
                navController.navigate("${Screen.Details.name}/$username")
            }
        }
        composable("${Screen.Details.name}/{user}") { backStackEntry ->
            val user = backStackEntry.arguments?.getString("user") ?: "Default"
            DetailsScreen(user)
        }
    }
}

@Composable
fun DetailsScreen(user: String) {
    Box(
        contentAlignment = Alignment.Center, // 设置内容在 Box 中居中
        modifier = Modifier.fillMaxSize() // Box 占满整个屏幕
    ) {
        Text("This is DetailsScreen of User: \n$user")
    }
}

@Composable
fun HomeScreen(
    onButtonClicked: (String)->Unit,
){
    var user by remember { mutableStateOf("")}
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
    ) {
        OutlinedTextField(
            value = user,
            onValueChange = { user = it },
            label = {Text("Where to go?")},
            singleLine = true
        )
        Spacer1(Modifier.height(24.dp))
        Button(
            onClick = {onButtonClicked(user)}
        ) { Text("Details")}
    }
}

```

