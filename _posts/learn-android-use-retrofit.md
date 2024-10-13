---
title: 学习笔记：在 Android 中使用 Retrofit 访问互联网
date: 2024-10-09 18:30:10hs
tags:
 - Android
 - Retrofit
 - Hilt
categories:
 - Note
---

Ref:
[从互联网获取数据](https://developer.android.com/courses/pathways/android-basics-compose-unit-5-pathway-1?hl=zh-cn)
[加载并显示来自互联网的图片](https://developer.android.com/courses/pathways/android-basics-compose-unit-5-pathway-2?hl=zh-cn)

参考上述两个教程，从 0 实现一个使用 Retrofit 连接 REST Web 服务的 APP 。  
使用：
- MVVM 架构
- 仓库模式
- 依赖注入

以下网址将获取火星照片列表：  
https://android-kotlin-fun-mars-server.appspot.com/photos 

```json
[
  {
    "id": "424905",
    "img_src": "https://mars.jpl.nasa.gov/msl-raw-images/msss/01000/mcam/1000MR0044631300503690E01_DXXX.jpg"
  },
  //...
]
```
<!--more-->

## 实现 APP 的界面

### 新建项目

使用 Android Studio 版本 ： **Ladybug 2024.2.1**

`File` > `New Project` > 选择 `Empty Activity` 

![New Project](https://m.nep.me/blog/post/le-retrofit-new-porject.jpg)

包名为 `com.example.marsphoto`

创建完成后，移除 Greeting 函数，`MainActivity.kt` 的内容如下
```kotlin 
package com.example.marsphoto

import ...

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MarsPhotoTheme {
                MarsPhotosApp()
                }
            }
        }
    }


@Composable
fun MarsPhotosApp(){

}
```

### 添加依赖(viewModel 与 Retrofit)

需要添加下面的依赖至 `build.gradle.kts (Module:app)`
```kotlin
// Retrofit
implementation("com.squareup.retrofit2:retrofit:2.9.0")
// Retrofit with Scalar Converter 
// 将 Json 转为字符串，访问 Json 测试使用，然后更换 kotlinx.serialization 
implementation("com.squareup.retrofit2:converter-scalars:2.9.0")

// lifecycle viewModel
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
```

使用 version catalogs
`libs.versions.toml`
```t
[versions]
retrofit = "2.9.0"
lifecycleRuntimeKtx = "2.8.6"

[libraries]
# lifecycle-viewmodel-compose
androidx-lifecycle-viewmodel-compose = { module = "androidx.lifecycle:lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
# retrofit
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
converter-scalars = { module = "com.squareup.retrofit2:converter-scalars", version.ref = "retrofit" }
```

`build.gradle.kts (Module:app)`
```kotlin
// Retrofit
implementation(libs.retrofit)
// Retrofit with Scalar Converter
// 将 Json 转为字符串，访问 Json 测试使用，然后更换 kotlinx.serialization
implementation(libs.converter.scalars)
// lifecycle viewModel
implementation(libs.androidx.lifecycle.viewmodel.compose)
``` 

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/baa812ef7f0b992cb8c00d6c60b88c0d335ddfaf#diff-4db78a6b5a12c48c1fe1e51608c79dc8af7124b324c512eea41baea0fa29b990R55)

### 创建 Screen 与 viewModel 文件
分别创建 `HomeScreen.kt` 与 `MarsViewModel.kt` 文件，以固定文本代替 Photos 的获取结果

`MarsViewModel.kt`
```kotlin
package com.example.marsphoto.ui

import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.compose.runtime.getValue
import androidx.lifecycle.ViewModel

class MarsViewModel: ViewModel(){
    var marsUiState: String by mutableStateOf("")
        private set

    init{
        getMarsPhotos()
    }

    private fun getMarsPhotos() {
        marsUiState = "Get Photos with Mars Api"
    }
}
```

`HomeScreen.kt`
```kotlin 
package com.example.marsphoto.ui

import ...

@Composable
fun HomeScreen(
    modifier: Modifier = Modifier
) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier.fillMaxSize().verticalScroll(rememberScrollState())
    ) {
        val viewModel: MarsViewModel = viewModel()
        val photos = viewModel.marsUiState
        Text(text = photos)
    }
}
```

在 `MainActivity.kt` 调用 `HomeScreen()` ，运行后可以查看 `Get Photos with Mars Api` 显示在屏幕中
```kotlin 
@Composable
fun MarsPhotosApp(){
    HomeScreen()
}
```

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/6d4262243c2e04f3eef7d61de231f435db6d63bb)

## 使用 Retrofit 访问网络
### 为 APP 添加联网权限

打开 `manifests/AndroidManifest.xml` 在 <application> 前面添加
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### 创建 Retrofit 对象  
创建 `network` 包，创建 `MarsApiService.kt` 文件，创建 Retrofit 对象  

```kotlin 
// 设置 BASE_URL
private const val BASE_URL =
    "https://android-kotlin-fun-mars-server.appspot.com"

//创建 Retrofit 对象
private val retrofit = Retrofit.Builder()
    .addConverterFactory(ScalarsConverterFactory.create())
    .baseUrl(BASE_URL)
    .build()

//定义 MarsApiService 的接口，该接口定义 Retrofit 如何使用 HTTP 请求与网络服务器通信。
interface MarsApiService{
    @GET("photos")
    suspend fun getPhotos(): String
}

// 声明单例对象。单例模式可确保对于一个对象只创建一个实例。
object MarsApi{
    val retrofitService: MarsApiService by lazy {
        retrofit.create(MarsApiService::class.java)
    }
}
```
**MarsApi** 对象通过 **object** 关键字声明，确保在应用程序的生命周期内只创建一个 **MarsApiService** 实例。  
这种方式有助于管理网络请求的一致性。    
单例模式应该只用于测试，正式使用推荐 [Android 中的依赖项注入](https://developer.android.com/training/dependency-injection?hl=zh-cn)


### 在 viewModel 中调用 Retrofit 对象  
修改 `MarsViewModel.kt` 的 `getMarsPhotos()` ，将硬编码的 `marsUiState = "Get Photos with Mars Api"` 修改为下面的内容 

```kotlin
private fun getMarsPhotos() {
        viewModelScope.launch{
            val listResult = MarsApi.retrofitService.getPhotos()
            marsUiState = listResult
        }
    }
```

重新编译即可查看 JSON 的内容显示在屏幕中。 

## 使用 kotlinx.serialization 解析 JSON 
### 添加依赖
编辑 `build.gradle.kts (Module :app)`

在 `plugins` 代码块中，添加 `kotlinx serialization` 插件。 
```kotlin 
id("org.jetbrains.kotlin.plugin.serialization") version "1.9.22"
```
在 `dependencies` 部分添加 
```kotlin 
// Kotlin serialization
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3")
```

将 Scalar Converter 替换为 Kotlin serialization Converter
```kotlin 
// Retrofit with Kotlin serialization Converter
implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
implementation("com.squareup.okhttp3:okhttp:4.11.0")
```

### 创建 Kotlin 对象

在 network 下创建 `MarsPhoto.kt` 数据类，包含一个 id 与一个 imgSrc 属性。
```kotlin 
package com.example.marsphoto.network

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class MarsPhoto(
    val id: String,
    @SerialName(value = "img_src")
    val imgSrc: String
)
```
Json 中的 id 是 "" 封装的，所以是 `String` 而不是 `Int`

`@Serializable` 注解表示类可序列化，kotlinx serialization 解析 JSON 时，它会按名称匹配键。
`@SerialName` 注解，如果 json 中的名称格式不符合 kotlin 的规范，可以使用这个注解映射， `@SerialName(value = "img_src")` 将该变量映射到 JSON 属性 `img_src`。

### 更新 Retrofit 与 MarsViewModel
更新 Retrofit 的构建起，使用 kotlinx.serialization 。  
编辑 `MarsApiService.kt`

```kotlin
private val retrofit = Retrofit.Builder()
    .addConverterFactory(ScalarsConverterFactory.create())
```
修改为
```kotlin
private val retrofit = Retrofit.Builder()
    .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
```

现在可以要求 Retrofit 返回 `MarsPhoto` 对象，而不是 `String` 。

```kotlin
interface MarsApiService{
    @GET("photos")
    suspend fun getPhotos(): List<MarsPhoto>
}
```

由于 Retrofit 不再返回 String 类型，MarsViewModel 中调用部分也需要修改。

```
marsUiState = "Get Photos with Mars Api"
// to
marsUiState = "There are ${listResult.size} MarPhotos"
```

运行查看结果。

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/f8f98df565c40193e2f09fa44ad31a532dd1b2b5)

## 添加 Loading 与 Error 屏幕

改造一下 MarsViewModel 与 HomeScreen ，使 APP 可以显示 Loading 与 Error 的情况。
HomeScreen
```kotlin

@Composable
fun HomeScreen(
    modifier: Modifier = Modifier
) {
    val viewModel: MarsViewModel = viewModel()
    val marsUiState = viewModel.marsUiState
    when(marsUiState){
        is MarsUiState.Success -> SuccessScreen(
            marsUiState = marsUiState,
            modifier = modifier.fillMaxSize().verticalScroll(rememberScrollState())
        )
        MarsUiState.Loading -> LoadingScreen(modifier = modifier.fillMaxSize())
        MarsUiState.Error -> ErrorScreen(modifier = modifier.fillMaxSize())
    }
}

@Composable
fun LoadingScreen(modifier: Modifier = Modifier) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
    ) {
        Text("Loading")
    }
}

@Composable
fun ErrorScreen(modifier: Modifier = Modifier) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
    ) {
        Text("Meet Error")
    }
}

@Composable
fun SuccessScreen(
    marsUiState: MarsUiState,
    modifier: Modifier = Modifier
){
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
    ) {
        Text(text = marsUiState.toString())
    }
}
```
MarsViewModel
```kotlin
sealed interface MarsUiState{
    data class Success(val photo: String): MarsUiState
    object Error: MarsUiState
    object Loading: MarsUiState
}

class MarsViewModel: ViewModel(){
    var marsUiState: MarsUiState by mutableStateOf(MarsUiState.Loading)
        private set

    init{
        getMarsPhotos()
    }

    private fun getMarsPhotos() {
        viewModelScope.launch{
            marsUiState = MarsUiState.Loading
            marsUiState = try {
                val listResult = MarsApi.retrofitService.getPhotos()
                MarsUiState.Success(
                    "Success: ${listResult.size} Mars photos retrieved"
                )
            } catch (_: IOException){
                MarsUiState.Error
            } catch (_: HttpException){
                MarsUiState.Error
            }
        }
    }
}
```

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/1df4882ec47ebb63be5318de671b21caed2c3734)

## 添加仓库
### 数据层与界面层
根据 [Android 的推荐应用架构](https://developer.android.com/topic/architecture?hl=zh-cn#recommended-app-arch) ，应用应至少具有一个界面层和一个数据层。

>数据层负责应用的业务逻辑以及为应用寻源和保存数据。数据层使用单向数据流模式向界面层公开数据。数据可能来自多个来源，例如网络请求、本地数据库或设备上的文件。
>一个应用甚至可能有多个数据源。
>界面层的关注点是显示所提供的数据。界面(Screen 与 ViewModel)不再检索数据，因为这是数据层的关注点。
>数据层由一个或多个仓库组成。仓库本身包含零个或多个数据源。

仓库类的作用包括：
 - 向应用的其余部分公开数据。
 - 集中管理数据更改。
 - 解决多个数据源之间的冲突。
 - 对应用其余部分的数据源进行抽象化处理。
 - 存放业务逻辑。

Mars Photos 应用只有一个数据源，即网络 API 调用。它只检索数据，因此没有任何业务逻辑。数据通过仓库类公开提供给应用，该类会对数据源进行抽象化处理。

![Layer](https://m.nep.me/blog/post/le-retrofit-data-layer.png)

### 创建仓库
Android 开发者指南指出，仓库类以其所负责的数据命名。仓库命名惯例是 **数据类型** + **仓库**。在这个应用中，其名称为 **MarsPhotosRepository**。

- 创建 package `data`
- 新建一个名称为 `MarsPhotosRepository.kt` 的 **Interface** 
```kotlin
package com.example.marsphoto.data

import com.example.marsphoto.network.MarsApiService
import com.example.marsphoto.network.MarsPhoto

interface MarsPhotosRepository {
    suspend fun getMarsPhotos(): List<MarsPhoto>
}
```
- 然后创建一个名称为 `NetworkMarsPhotosRepository` 的类来实现 MarsPhotosRepository 接口
```kotlin
class NetWorkMarsPhotosRepository : MarsPhotosRepository{
    override suspend fun getMarsPhotos(): List<MarsPhoto> {
        return MarsApi.retrofitService.getPhotos()
    }
}
```
编辑 `MarsViewModel.kt` 文件，将 `val listResult = MarsApi.retrofitService.getPhotos()` 行替换为以下代码：
```kotlin
val marsPhotosRepository = NetWorkMarsPhotosRepository()
val listResult = marsPhotosRepository.getMarsPhotos()
```
运行应用，查看结果，确认与之前相同。  
仓库将提供数据，而不是由 ViewModel 直接发出关于数据的网络请求。ViewModel 不再直接引用 MarsApi 代码。
![ViewModel without MarsApi](https://m.nep.me/blog/post/le-retrofit-viewmdel-without-marsapi.png)

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/30b4457fa30487e984bcddc13d00cd562078ff2c)

## 手动依赖项注入(DI) 
Ref: [手动依赖项注入](https://developer.android.com/codelabs/basic-android-kotlin-compose-add-repository?hl=zh-cn#3)  
依赖项注入（Dependency Injection，DI） 是一种软件设计模式，旨在将组件之间的依赖关系从内部创建转移到外部提供。这意味着一个类不再负责创建它所依赖的对象，而是通过构造函数、属性或方法参数由外部提供这些依赖。

**核心概念：**
- **依赖项（Dependencies）**： 一个类所需要的其他类或对象。
- **注入（Injection）**： 通过外部手段（如构造函数、属性、方法）将依赖项提供给需要的类。

**依赖项注入的主要好处包括：**  
- **解耦合**： 类与其依赖项之间的耦合度降低，增强模块间的独立性。
- **可测试性**： 更容易为类编写单元测试，因为可以轻松地替换依赖项为模拟对象。
- **可维护性**： 更清晰的依赖关系使得代码更易于理解和维护。
- **灵活性与可扩展性**： 依赖项的实现可以在不改变依赖方代码的情况下进行替换或扩展。

### 手动依赖项注入的方法 
**构造函数注入** 是最常见且推荐的 DI 方法。通过在类的构造函数中声明依赖项，使得这些依赖项在创建类的实例时被提供。

示例：
```kotlin 
class Repository {
    fun getData(): String = "Hello from Repository"
}

class ViewModel(private val repository: Repository) {
    fun fetchData(): String = repository.getData()
}
```
使用：
```kotlin
val repository = Repository()
val viewModel = ViewModel(repository)
```

**单例模式与服务定位器模式**
**单例模式** 和 **服务定位器模式** 是另一种手动 DI 的方法，但需要谨慎使用，因为它们可能引入全局状态和隐式依赖，降低代码的可测试性。
```kotlin
object ServiceLocator {
    val repository: Repository by lazy { Repository() }
}

class ViewModel {
    private val repository = ServiceLocator.repository
    fun fetchData(): String = repository.getData()
}
```
这种方法使得依赖项在类中是隐式的，不利于测试和维护。

### 创建应用容器
- 在 `data` 下创建一个 `AppContainer` 的 **Interface**
- 在 `AppContainer` 内添加一个名为 `marsPhotosRepository` 且类型为 `MarsPhotosRepository` 的抽象属性。
```kotlin
interface AppContainer {
    val marsPhotosRepository: MarsPhotosRepository
}
```
- 在接口定义下方，创建一个名为 `DefaultAppContainer` 的类来实现 `AppContainer` 接口。
- 将 `network/MarsApiService.kt` 中的内容移至 `DefaultAppContainer` 类，让它们都位于用于维护依赖项的容器中。
MarsApiService.kt 只需要保留如下内容
```kotlin
interface MarsApiService{
    @GET("photos")
    suspend fun getPhotos(): List<MarsPhoto>
}
```
- `DefaultAppContainer` 类会实现 `AppContainer` 接口，因此我们需要替换 `marsPhotosRepository` 属性。在变量 `retrofitService` 后面添加以下代码：
```kotlin
override val marsPhotosRepository: MarsPhotosRepository by lazy {
    NetworkMarsPhotosRepository(retrofitService)
}
```
完成的 `DefaultAppContainer` 类应如下所示：  
```kotlin 
package com.example.marsphoto.data

import ...

interface AppContainer {
    val marsPhotosRepository: MarsPhotosRepository
}

class DefaultAppContainer: AppContainer{
    private val baseUrl = "https://android-kotlin-fun-mars-server.appspot.com"

    private val retrofit = Retrofit.Builder()
        .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
        .baseUrl(baseUrl)
        .build()
    
    /**
     * Retrofit service object for creating api calls
     */
    private val retrofitService: MarsApiService by lazy {
        retrofit.create(MarsApiService::class.java)
    }

    /**
     * DI implementation for Mars photos repository
     */
    override val marsPhotosRepository: MarsPhotosRepository by lazy {
        NetWorkMarsPhotosRepository(retrofitService)
    }
}
```
- 在 `data/MarsPhotosRepository.kt` 需要编辑 `NetworkMarsPhotosRepository` 传递 `retrofitService`
```kotlin
class NetWorkMarsPhotosRepository(private val marsApiService: MarsApiService) : MarsPhotosRepository{
    override suspend fun getMarsPhotos(): List<MarsPhoto> = marsApiService.getPhotos()
}
```
### 将应用容器附加到应用

目的：将应用容器与应用的生命周期关联起来，使得整个应用都能访问到这些依赖项。

在 `com.example.marsphotos` 下创建类 `MarsPhotosApplication` ， 此类继承自 `Application()` 
```kotlin
import android.app.Application

class MarsPhotosApplication : Application() {
}
```

在 `MarsPhotosApplication` 类中，声明一个名为 `container` 且类型为 `AppContainer` 的变量，用于存储 `DefaultAppContainer` 对象。该变量会在调用 `onCreate()` 期间初始化，因此需要使用 `lateinit` 修饰符标记该变量。
```kotlin
package com.example.marsphotos

import android.app.Application
import com.example.marsphotos.data.AppContainer
import com.example.marsphotos.data.DefaultAppContainer

class MarsPhotosApplication : Application() {
    lateinit var container: AppContainer
    override fun onCreate() {
        super.onCreate()
        container = DefaultAppContainer()
    }
}
```

编辑清单文件 `manifests/AndroidManifest.xml` ，添加值为应用类名称 ".MarsPhotosApplication" 的 android:name 属性。```xml
```xml
<application
   android:name=".MarsPhotosApplication"
   android:allowBackup="true"
   ... >
...
</application>
```

### 将仓库添加到 ViewModel
完成上述步骤后，ViewModel 可以调用仓库对象来检索数据。  

编辑 `MarsViewModel.kt` 文件，在 `MarsViewModel` 的类声明中，添加一个类型为 `MarsPhotosRepository` 的私有构造函数形参 `marsPhotosRepository`。构造函数形参的值来自应用容器，因为应用现在在使用依赖项注入。  

```kotlin
class MarsViewModel(private val marsPhotosRepository: MarsPhotosRepository) : ViewModel(){
    //...
}
```

在 `getMarsPhotos()` 函数中可以移除 
```kotlin
val marsPhotosRepository = NetworkMarsPhotosRepository()
```

由于 Android 框架不允许在创建时向构造函数中的 ViewModel 传递值，因此我们实现了一个 ViewModelProvider.Factory 对象来绕过此限制。

在函数 `MarsViewModel.kt` 的 `getMarsPhotos()` 下，输入伴生对象的代码。
```kotlin
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.ViewModelProvider.AndroidViewModelFactory.Companion.APPLICATION_KEY
import androidx.lifecycle.viewModelScope
import androidx.lifecycle.viewmodel.initializer
import androidx.lifecycle.viewmodel.viewModelFactory
import com.example.marsphoto.MarsPhotosApplication

companion object {
   val Factory: ViewModelProvider.Factory = viewModelFactory {
       initializer {
           val application = (this[APPLICATION_KEY] as MarsPhotosApplication)
           val marsPhotosRepository = application.container.marsPhotosRepository
           MarsViewModel(marsPhotosRepository = marsPhotosRepository)
       }
   }
}
```

在 `HomeScreen.kt` 文件中，将
```kotlin 
val viewModel: MarsViewModel = viewModel()
```
修改为
```kotlin 
val viewModel: MarsViewModel = viewModel(factory = MarsViewModel.Factory)
```

运行应用，确认应用是否像之前一样正常运行。  
它现已使用仓库和依赖项注入！通过仓库实现数据层之后，界面和数据源已实现分离，并且符合 Android 最佳实践。

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/629e4ae7c92cc8634340ce893077460d16d756cf)
[学习设置测试](https://developer.android.com/codelabs/basic-android-kotlin-compose-add-repository?hl=zh-cn#6)

### 总结  
**应用容器** 用于集中管理和提供应用所需的各种依赖项（如仓库、网络服务等）

1. **Repositroy 提供数据**
Repository 定义数据获取和处理逻辑，并将其提供给应用的其他部分（如 ViewModel）。
2. **在 AppContainer 中定义**
声明应用所需的依赖项。
```kotlin
interface AppContainer {
    val marsPhotosRepository: MarsPhotosRepository
}
```
3.**在 DefaultAppContainer 中实现注入** 
具体创建和提供依赖实例。  
`DefaultAppContainer` 类实现了 `AppContainer` 接口，并具体提供了各个依赖项的实例。
4.**在 Application() 中初始化**
在应用启动时初始化依赖容器。  
```kotlin
class MarsPhotosApplication : Application() {
    lateinit var container: AppContainer
    
    override fun onCreate() {
        super.onCreate()
        container = DefaultAppContainer()
    }
}
``` 
同时需要修改清单文件
```xml
<application
    android:name=".MarsPhotosApplication"
    android:allowBackup="true"
    ... >
    ...
</application>
```
5. **通过构造函数传递给 ViewModel** 
将依赖注入到 ViewModel 中。  
```kotlin
class MarsViewModel(private val marsPhotosRepository: MarsPhotosRepository) : ViewModel() {
    // ViewModel 的实现
}
```
6. **需要自定义 ViewModel.Factory 实现参数的传递** 
创建自定义工厂以支持依赖注入。  
```kotlin
companion object {
    val Factory: ViewModelProvider.Factory = viewModelFactory {
        initializer {
            val application = (this[APPLICATION_KEY] as MarsPhotosApplication)
            val marsPhotosRepository = application.container.marsPhotosRepository
            MarsViewModel(marsPhotosRepository = marsPhotosRepository)
        }
    }
}
```
7. **在 Screen @Composable 函数中使用 Factory 初始化 ViewModel**
在 UI 层获取并使用 ViewModel。  
```kotlin
@Composable
fun HomeScreen() {
    val viewModel: MarsViewModel = viewModel(factory = MarsViewModel.Factory)
    // 使用 ViewModel 进行 UI 操作
}
```

## 添加测试
### 测试依赖
**libs.versions.toml**  
```t 
[versions]
# junit Android Studio 2024.2.1 默认项目自带
junit = "4.13.2"
# kotlinxCoroutinesTest
kotlinxCoroutinesTest = "1.8.1"

[libraries]
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "junitVersion" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinxCoroutinesTest" }
```

`build.gradle.kts (Module: app)` > `dependencies`
```kotlin
testImplementation(libs.junit)
testImplementation(libs.kotlinx.coroutines.test)
```

### 添加测试数据
在 com.example.marsphoto(test) 目录下创建 package `fake` ，创建 `FakeDataSource` 文件，填入下面的内容
(androidTest) 是需要编译安装至模拟器进行测试
```kotlin
object FakeDataSource {
    private const val idOne = "img1"
    private const val idTwo = "img2"
    private const val imgOne = "url.one"
    private const val imgTwo = "url.two"
    val photosList = listOf(
        MarsPhoto(
            id = idOne,
            imgSrc = imgOne
        ),
        MarsPhoto(
            id = idTwo,
            imgSrc = imgTwo
        )
    )
}
```

### 进行测试
**创建 FakeMarsApi.kt**
```kotlin 
class FakeMarsApiService: MarsApiService{
    override suspend fun getPhotos(): List<MarsPhoto> = FakeDataSource.photosList
}
```

**创建 NetworkMarsRepositoryTest**，填入测试内容
```kotlin
class NetworkMarsRepositoryTest {
    @Test
    fun networkMarsPhotoRepository_getMarsPhotos_verifyPhotoList() =
        runTest {
            val repository = NetWorkMarsPhotosRepository(
                marsApiService = FakeMarsApiService()
            )
                assertEquals(FakeDataSource.photosList, repository.getMarsPhotos())
        }
}
```

### 测试 ViewModel
创建 **FakeNetworkMarsPhotosRepository** 类供测试使用
```kotlin
class FakeNetworkMarsPhotosRepository : MarsPhotosRepository{
    override suspend fun getMarsPhotos(): List<MarsPhoto> = FakeDataSource.photosList
}
```

创建，MarsViewModelTest 新类，编写 ViewModel 测试 
```kotlin
class MarsViewModelTest (){
    @Test
    fun marsViewModel_getMarsPhotos_verifyMarsUiStateSuccess() =
        runTest{
            val viewModel = MarsViewModel(
                marsPhotosRepository = FakeNetworkMarsPhotosRepository()
            )
            assertEquals(
                MarsUiState.Success("Success: ${FakeDataSource.photosList.size} Mars photos retrieved"),
                viewModel.marsUiState
            )
        }
}
```

运行测试后报错，出现错误的原因是 Android 界面线程在单元测试中不可用，可以移到 androidTest 中进行测试。  
不过依然报错
```
junit.framework.AssertionFailedError: expected:<Success(photo=Success: 2 Mars photos retrieved)> but
was:<com.example.marsphoto.ui.MarsUiState$Loading@93ca461>
```
这是因为 ViewModel 中的 `getMarsPhotos()` 函数是异步的，而测试用例没有等待异步任务完成。  

可以在断言前，添加 `advanceUntilIdle()` 让测试等待协程执行完毕
`MarsViewModelAndroidTest.kt`
```kotlin
class MarsViewModelTest (){
    @OptIn(ExperimentalCoroutinesApi::class)
    @Test
    fun marsViewModel_getMarsPhotos_verifyMarsUiStateSuccess() =
        runTest {
            val viewModel = MarsViewModel(
                marsPhotosRepository = FakeNetworkMarsPhotosRepository()
            )

            //让测试等待协程执行完毕
            advanceUntilIdle()

            assertEquals(
                MarsUiState.Success("Success: ${FakeDataSource.photosList.size} Mars photos retrieved"),
                viewModel.marsUiState
            )
        }
}
```
再使用 androidTest 就显示通过。

当然在普通的单元测试中也是可以进行测试的，要在运行单元测试时 [明确定义默认调度程序](https://developer.android.com/codelabs/basic-android-kotlin-compose-add-repository?hl=zh-cn#9) `#创建测试调度程序`。   

1. 在测试目录中创建一个名为 `rules` 的新软件包。
2. 在 `rules` 目录中，新建一个名为 `TestDispatcherRule` 的类。
3. 使用 `TestWatcher` 扩展 `TestDispatcherRul`e。借助 TestWatcher 类，可以在测试的不同执行阶段执行操作。
4. 为 `TestDispatcherRule` 创建一个 `TestDispatcher` 构造函数形参。
5. 替换 `starting()` 函数，在测试开始执行之前将 Main 调度程序替换为测试调度程序，添加对 `Dispatchers.setMain()` 的调用，并传入 `testDispatcher` 作为实参。
6. 替换 `finished()` 方法以重置 Main 调度程序。调用 `Dispatchers.resetMain()` 函数。
```kotlin
class TestDispatcherRule(
    val testDispatcher: TestDispatcher = UnconfinedTestDispatcher(),
) : TestWatcher() {
    override fun starting(description: Description) {
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```
7. 在 `MarsViewModelTest` 类中，对 `TestDispatcherRule` 类进行实例化并将其分配给 `testDispatcher` 只读属性。应用于测试时需要将 `@get:Rule` 注解添加到 `testDispatcher` 属性。
```kotlin
class MarsViewModelTest {
    @get:Rule
    val testDispatcher = TestDispatcherRule()
    ...
}
```
再运行就可以了，教程是这么写的
但是我运行提示 `java.lang.NoClassDefFoundError: com/example/marsphoto/ui/MarsViewModel` ，下个 Github 的版本，直接运行也是这个报错。
//TODO   

## 使用 Hilt 注入
### 添加依赖
### 使用 version catalogs 添加 Hilt 依赖(Android Studio 2024.2.1)
使用 version catalogs

编辑 **libs.versions.toml** 
```t
[versions]
kotlin="2.0.20"
ksp="2.0.20-1.0.24"
hilt="2.51.1"
hiltCommon="1.2.0"

[libraries]
# hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-android-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
# hilt-common
androidx-hilt-common = { group = "androidx.hilt", name = "hilt-common", version.ref = "hiltCommon" }
androidx-hilt-compiler = { group = "androidx.hilt", name = "hilt-compiler", version.ref = "hiltCommon" }
androidx-hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hiltCommon" }

[plugins]
# ksp
google-ksp = { id = "com.google.devtools.ksp", version.ref = "ksp"}
android-hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt"}

[bundles]
hilt = ["hilt-android", "androidx-hilt-common", "androidx-hilt-navigation-compose"]
hilt-ksp = ["hilt-android-compiler", "androidx-hilt-compiler"]
```

`build.gradle.kts (Project)` > `plugins` 添加
```kotlin
alias(libs.plugins.google.ksp) apply false
alias(libs.plugins.android.hilt) apply false
```

在 `build.gradle.kts (Module: app)` > `plugins` 部分添加
```kotlin
alias(libs.plugins.google.ksp)
alias(libs.plugins.android.hilt)
```

在 `build.gradle.kts (Module: app)` > `dependencies` 部分添加
```kotlin
// Hilt
implementation(libs.bundles.hilt)
ksp(libs.bundles.hilt.ksp)
```

如果 Android Studio sync 之后全线标红报错，只要在 `build.gradle.kts (Module: app)` 输入一行再撤销就好了。 

完成可以 build 一下 APP ，看看是否会报错。  

Ref: [Version Catalogを使ってみましょう](https://engineering.nifty.co.jp/blog/26566)

### 使用 hilt 

除了依赖之外的起始代码为之前添加完了 `MarsPhotosRepository.kt` 的代码

1. 首先创建 `MarsApplication` 类继承自 `Application()` 并添加 `@HiltAndroidApp` 注解
所有使用 Hilt 的应用都必须包含一个带有 `@HiltAndroidApp` 注解的 `Application()` 类
```kotlin
@HiltAndroidApp
class MarsApplication : Application(){
    //在这里初始化全局依赖项
}
```
与手动注入一样，需要在清单文件 `AndroidManifest.xml` 中声明创建的 Application 类：
```xml
<application
    android:name=".MarsApplication"
    ... >
    <!-- 其他配置 -->
</application>
```

2. 创建 Hilt 模块以提供依赖项
为了让 Hilt 知道如何提供 `Retrofit`、`MarsApiService` 和 `MarsPhotosRepository`，我们需要创建一个模块。模块是一个用 `@Module` 注解的类，里面包含了提供依赖的方法，用 `@Provides` 或 `@Binds` 注解标注。  
创建一个 `di` 包(package)，在其中创建 `AppModule.kt` 
```kotlin
package com.example.marsphoto.di

import com.example.marsphoto.data.MarsPhotosRepository
import com.example.marsphoto.data.NetWorkMarsPhotosRepository
import com.example.marsphoto.network.MarsApiService
import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import dagger.Binds
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import kotlinx.serialization.json.Json
import retrofit2.Retrofit
import javax.inject.Singleton

private const val BASE_URL = "https://android-kotlin-fun-mars-server.appspot.com"

@Module
@InstallIn(SingletonComponent::class)
abstract class AppModule{

    // 使用 @Binds 将接口绑定到实现
    @Binds
    @Singleton
    abstract fun bindMarsPhotosRepository(
        netWorkMarsPhotosRepository: NetWorkMarsPhotosRepository
    ) : MarsPhotosRepository

    companion object {
        // 使用 @Provides 提供 Retrofit 实例
        @Provides
        @Singleton
        fun provideRetrofit(): Retrofit {
            return Retrofit.Builder()
                .addConverterFactory(Json.asConverterFactory("application/json".toMeditaType()))
                .baseUrl(BASE_URL)
                .build()
        }

        // 使用 @Provides 提供 MarsApiService 实例
        @Provides
        @Singleton
        fun provideMarsApiService(retrofit: Retrofit) : MarsApiService{
            return retrofit.create(MarsApiService::class.java)
        }
    }
}
```
 - `@Module`：标记该类为 `Hilt` 模块。
 - `@InstallIn(SingletonComponent::class)`：指定模块的生命周期，这里使用 `SingletonComponent` 使其在整个应用程序生命周期内有效。
 - `@Binds`：用于绑定接口到实现。在这种情况下，我们将 `MarsPhotosRepository` 接口绑定到 `NetWorkMarsPhotosRepository` 实现。
 - `@Provides`：用于提供具体的依赖项实例，如 `Retrofit` 和 `MarsApiService`

3. 修改 MarsApiService.kt
与手动注入一样，`MarsApiService.kt` 只需要保留接口的定义，不再需要 MarsApi 对象。
```kotlin
package com.example.marsphoto.network

import retrofit2.http.GET

interface MarsApiService{
    @GET("photos")
    suspend fun getPhotos(): List<MarsPhoto>
}
```

4. 修改 `NetWorkMarsPhotosRepository.kt`
确保 `NetWorkMarsPhotosRepository` 使用依赖注入来获取 `MarsApiService`
```kotlin
class NetWorkMarsPhotosRepository @Inject constructor(
    private val marsApiService: MarsApiService
) : MarsPhotosRepository{
    override suspend fun getMarsPhotos(): List<MarsPhoto> = marsApiService.getPhotos()
}
```
`@Inject` 构造函数注解：告诉 Hilt 如何创建 `NetWorkMarsPhotosRepository`，即通过注入 `MarsApiService`。

5. 修改 `MarsViewModel.kt` 以使用 Hilt 进行依赖注入
```kotlin
class MarsViewModel: ViewModel() { 
    //...
}
```
修改为
```kotlin
@HiltViewModel 
class MarsViewModel @Inject constructor(
    private val marsPhotosRepository: MarsPhotosRepository
) : ViewModel(){
    //...
}
```
移除 `getMarsPhotos()` 函数中的 
```kotlin
val marsPhotosRepository = NetWorkMarsPhotosRepository()
```
 - `@HiltViewModel`：标记 ViewModel 以便 Hilt 可以为其提供依赖项。
 - `@Inject constructor{}`：注入 MarsPhotosRepository 到 ViewModel 中。
 - 移除了直接创建 `NetWorkMarsPhotosRepository` 的代码，改为使用注入的 `marsPhotosRepository。`

6. 修改 UI Screen 中的 `viewModel()` 为 `hiltViewModel()`
```kotlin
val viewModel: MarsViewModel = hiltViewModel()
```

7. 修改 Activity 以使用 Hilt
确保使用 viewModel 的 Activity 或 Fragment 使用 `@AndroidEntryPoint` 注解，以便 Hilt 能够为它们提供依赖项。
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() { 
    //...
}
```
运行 APP ，可以正常显示结果。  

### 总结

在 `AppModule.kt` 中，使用了一个 `@Binds` 方法来将 `MarsPhotosRepository` 接口绑定到 `NetWorkMarsPhotosRepository`实现。
有关 `@Binds` 方法：
Hilt 在编译时会处理所有的模块和绑定方法，生成相应的代码来管理依赖关系。  
`@Binds` 作用是告诉 Hilt 如何将接口类型绑定到具体实现。  
当 `Hilt` 需要提供 `MarsPhotosRepository` 的实例时，它会参考 `@Binds` 方法的定义，知道应该实例化 `NetWorkMarsPhotosRepository` 并将其作为 `MarsPhotosRepository` 的实现。

Hilt 的依赖注入流程
```
MainActivity
    |
    |-- @Composable HomeScreen
            |
            |-- MarsViewModel
                    |
                    |-- MarsPhotosRepository (bound to NetWorkMarsPhotosRepository via @Binds)
                            |
                            |-- MarsApiService (provided by provideMarsApiService)
                                    |
                                    |-- Retrofit (provided by provideRetrofit)
```

[Github Commits](https://github.com/Homulilly/ComposeBase/commit/d13e81ad13d128d5d57a96f70c99c40f3cc53dcb)

## 使用 Coil 显示图片

### 添加依赖

```kotlin
implementation("io.coil-kt:coil-compose:2.7.0")
```

### Coil 示例
```kotlin 
AsyncImage(
    model = ImageRequest.Builder(LocalContext.current)
        .data("https://example.com/image.jpg")
        .crossfade(true)
        .build(),
    placeholder = painterResource(R.drawable.placeholder),
    contentDescription = stringResource(R.string.description),
    contentScale = ContentScale.Crop,
    modifier = Modifier.clip(CircleShape)
)
```

### 显示单张图片
在 `MarsViewModel.kt` 文件中，更新 `MarsUiState` 接口以接受 `MarsPhoto` 对象，而不是 `String`
```kotlin 
sealed interface MarsUiState{
    data class Success(val photo: MarsPhoto): MarsUiState
}
```

修改 `getPhotos()` 获取第一张图片
```kotlin
    private fun getMarsPhotos() {
        viewModelScope.launch{
            marsUiState = MarsUiState.Loading
            marsUiState = try {
                val photo = marsPhotosRepository.getMarsPhotos()[0]
                MarsUiState.Success(photo)
            } 
            //...
```

修改 HomeScreen.kt 显示图片，首先创建一个 `CoilScreen` 显示单张图片，
```kotlin
@Composable
fun CoilScreen(
    photo: MarsPhoto,
    modifier: Modifier = Modifier
){
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(photo.imgSrc)
            .crossfade(true)
            .build(),
        contentDescription = null,
        contentScale = ContentScale.FillWidth,
        modifier = modifier.fillMaxWidth().padding(8.dp)
    )
}
```

然后修改 `SuccessScreen` 接受的参数
```kotlin 
@Composable
fun SuccessScreen(
    photos: MarsPhoto,
    modifier: Modifier = Modifier) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
    ) {
        CoilScreen(photo)
    }
}
```

### 显示全部图片
在 `MarsViewModel.kt` 文件中，更新 `MarsUiState` 接口以接受 `List<MarsPhoto>` 对象
```kotlin 
sealed interface MarsUiState{
    data class Success(val photos: List<MarsPhoto>): MarsUiState
}
```

修改 `getPhotos()` 的获取 `List<MarsPhoto>` 对象
```kotlin
    private fun getMarsPhotos() {
        viewModelScope.launch{
            marsUiState = MarsUiState.Loading
            marsUiState = try {
                val photos = marsPhotosRepository.getMarsPhotos()
                MarsUiState.Success(photos)
            } 
            //...
```

然后修改 `SuccessScreen` 使用 `LazyColum` 显示列表 
```kotlin 
@Composable
fun SuccessScreen(
    photos: List<MarsPhoto>,
    modifier: Modifier = Modifier) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
    ) {
        LazyColumn {
            itemsIndexed(photos){ _, photo ->
                CoilScreen(photo)
            }
        }
    }
}
```

或是使用 `LazyVerticalGrid` 显示图片网格
