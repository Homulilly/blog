---
title: 学习笔记：Android Jetpack Compose 中的 Room 与 Hilt
date: 2024-10-13 11:51:43
tags:
 - Android
 - Room
 - Hilt
categories:
 - Note
---

## 介绍

Ref: *[使用 Android Jetpack 的 Room 部分将数据保存到本地数据库。](https://developer.android.com/training/data-storage/room?hl=zh-cn)*  
>**Room** 持久性库在 SQLite 上提供了一个抽象层，以便在充分利用 SQLite 的强大功能的同时，能够流畅地访问数据库。   
>三个主要组件：  
> - **数据库类**，用于保存数据库并作为应用持久性数据底层连接的主要访问点。
> - **数据实体**，用于表示应用的数据库中的表。
> - **数据访问对象(DAO)** ，为您的应用提供在数据库中查询、更新、插入和删除数据的方法。  

**Hilt** 是依赖注入库，由Google开发的，它是基于 Dagger 的一个扩展库，旨在简化 Android 应用程序中的依赖注入过程。Hilt 提供了一套注解和生成代码，用于在 Android 应用程序中自动完成依赖注入的配置。   

记录一下 Android Jetpack Compose 中使用 Room 与 Hilt 访问数据库的步骤。

Android Studio 版本 `Android Studio Ladybug | 2024.2.1`  

<!--more-->
## 步骤概览

- 添加相关依赖
- 使用 Room 的 `@Entity` 注解定义数据实体
- 使用 Room 的 `@Dao` 注解定义访问数据的方法
- 使用 Room 的 `@Database` 注解定义数据库
- 使用 Hilt 的 `@HiltAndroidApp` 注解创建 Application 

## 新建项目
在 Android Studio 中点击 `File` > `New Project`，选择 `Empty Activity` 

- **name**: `SimpleRoom`
- **Package name**: `com.example.simpleroom`

创建完成后，更新一下依赖的版本号

`libs.versions.toml`
```toml
kotlin = "2.0.20"  # "2.0.00"
composeBom = "2024.09.03" # "2024.04.01" 
```

## 添加依赖

在 Version Catalogs 的 `libs.versions.toml` 添加下面的内容

**[versions]** 部分：
```toml
lifecycleViewModelCompose="2.8.6"
ksp="2.0.20-1.0.24"
hilt="2.51.1"
hiltCommon="1.2.0"
room="2.6.1"
```

**[libraries]** 部分：
```toml
# viewModel Compose
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleViewModelCompose" }
# hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-android-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
androidx-hilt-common = { group = "androidx.hilt", name = "hilt-common", version.ref = "hiltCommon" }
androidx-hilt-compiler = { group = "androidx.hilt", name = "hilt-compiler", version.ref = "hiltCommon" }
androidx-hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hiltCommon" }
# room
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room"}
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room"}
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room"}
```

**[plugins]** 部分：
```toml
# ksp
google-ksp = { id = "com.google.devtools.ksp", version.ref = "ksp"} 
# hilt
android-hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt"}
```

增加一个 **[bundles]** 并添加：
```toml
[bundles]
hilt = ["hilt-android", "androidx-hilt-common", "androidx-hilt-navigation-compose"]
hilt-ksp = ["hilt-android-compiler", "androidx-hilt-compiler"]
room = ["androidx-room-runtime", "androidx-room-ktx"]
```

添加插件与库的依赖  
在 `build.gradle.kts (Project)` 的 `plugins` 部分添加：
```kotlin
alias(libs.plugins.google.ksp) apply false
alias(libs.plugins.android.hilt) apply false
```

在 `build.gradle.kts (Module :app)` 的 `plugins` 部分添加：
```kotlin
alias(libs.plugins.google.ksp) 
alias(libs.plugins.android.hilt) 
```

添加 
在 `build.gradle.kts (Module :app)` 的 `dependencies` 部分添加：

```kotlin
implementation(libs.androidx.lifecycle.viewmodel.compose)
implementation(libs.bundles.hilt)
implementation(libs.bundles.room)
ksp(libs.bundles.hilt.ksp)
ksp(libs.androidx.room.compiler)
```

sync 后 build 运行一下，查看是否报错。  

## Room

Ref: [Room Database with Hilt in Kotlin: A Guide to Store and Access Data in Android](https://rezaramesh.medium.com/room-database-with-hilt-in-kotlin-a-guide-to-store-and-access-data-in-android-c3001e507738)  

创一个用于保存 `User` 数据库，存储 `name` 与 `bio` 属性。

### 定义数据实体 

新建一个 `data` Package，在其中创建一个名为 `User` 的 data class ，并使用 `@Entity` 注解
```kotlin
package com.example.simpleroom.data  

@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val bio: String
)
```
`tableName` 设置数据库表的名称。   
`@PrimaryKey(autoGenerate = true)` 设置 id 为主键，并且从 0 开始自增。

下面的注解可以定义 name 为不可重复属性。
```kotlin
@Entity(
    tableName = "items",
    indices = [Index(value = ["name"], unique = true)]
)
```

### 创建数据访问对象（DAO）
[**数据访问对象 (DAO)**](https://developer.android.com/reference/androidx/room/Dao) 是一种模式，其作用是通过提供抽象接口将持久性层与应用的其余部分分离。

在 `data` 下新建一个名为 `UserDao` 的 interface ，需要添加 `@Dao` 注解 :
```kotlin 
package com.example.simpleroom.data 

@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.Companion.REPLACE)
    suspend fun insertUser(user: User)

    @Delete
    suspend fun deleteUser(user: User)

    @Update
    suspend fun updateUser(user: User)
    
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
}
```
`@Insert` `@Delete` `@Update` `@Query` 定义了增删改查方法。  

`@Insert(onConflict = OnConflictStrategy.Companion.REPLACE)` 定义了遇到冲突时的操作。  
`@Query()` 需要提供数据库语句。  

### 创建 Room 数据库类
创建 Room 数据库类需要创建一个抽象 `RoomDatabase` 类，并为其添加 `@Database` 注解。此类包含抽象方法用于提供 DAO 。  
在 `data` 下新建一个名为 `AppDatabase` 的抽象类:
```kotlin
package com.example.simpleroom.data

@Database(entities = [User::class], version = 1, exportSchema = false)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao() : UserDao
}
```

`@Database` 注解中
- `entities = [User::class]` 定义了包含的表，可以包含多个表
- `version = 1` 表示数据库版本，当数据库结构发生变化（如添加、删除或修改表、字段）时，需要更新版本号，并提供相应的迁移策略。
- `exportSchema` 是一个布尔类型的参数，用于指示 Room 是否应该将数据库的架构导出为一个 JSON 文件，默认为 `true`。

```kotlin
// 数据库包含多个表
@Database(entities = [User::class, Product::class], version = 2)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun productDao(): ProductDao
}
```

## 实现存储库(Repository)
**Repository** 模式提供了一个抽象层，用于处理数据操作，使得数据访问逻辑与其他部分的代码分离。  

新建一个名为 `repo` 的 Package，在其中创建一个名为 `UsersRepository` 的 interface ，需要实现与 `UserDao` 对应的函数。   
(如果只有一个数据源，可以不设置 interface ，直接创建 `UsersRepository` 类)：

```kotlin
package com.example.simpleroom.repo

import com.example.simpleroom.data.User
import kotlinx.coroutines.flow.Flow

interface UsersRepository {
    suspend fun insertUser(user: User)
    suspend fun updateUser(user: User)
    suspend fun deleteUser(user: User)
    
    fun getAllUsers(): Flow<List<User>>
}
```

在 `repo` 下，新建一个 `OfflineUsersRepository` 实现 `UsersRepository` ，并传入一个 `UserDao` 的参数（由于使用 **Hilt** 注入，所以需要使用 **@Inject constructor()** ）。  

```kotlin
package com.example.simpleroom.repo

class OfflineUsersRepository @Inject constructor (
    private val userDao: UserDao
) : UsersRepository{
    override suspend fun insertUser(user: User) = userDao.insertUser(user)

    override suspend fun updateUser(user: User) = userDao.updateUser(user)

    override suspend fun deleteUser(user: User) = userDao.deleteUser(user)

    override fun getAllUsers(): Flow<List<User>> = userDao.getAllUsers()
}
```

## 使用 Hilt 提供数据库实例

### 设置 Hilt 和 Application 类
新建一个继承自 `Application` 的 `RoomSimpleApplication` 的类，并使用 `@HiltAndroidApp` 注解

```kotlin
package com.example.simpleroom

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class RoomSimpleApplication : Application() {
}
```

在 AndroidManifest.xml 中声明应用程序类：
```xml
<application
    android:name=".RoomSimpleApplication"
    ... >
</application>
```

在 `MainActivity.kt` 添加 `@AndroidEntryPoint` 标注
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //...
```

### 创建 Hilt 模块  
新建一个名为 `di` 的 Package，在其中新建 `AppModule`： 
```kotlin
package com.example.simpleroom.di

@Module
@InstallIn(SingletonComponent::class)
abstract class AppModule {
    @Binds
    @Singleton
    abstract fun bindsUserRepository(
        offlineUsersRepository: OfflineUsersRepository
    ): UsersRepository

    companion object {
        @Provides
        @Singleton
        fun provideAppDatabase(@ApplicationContext context: Context) : AppDatabase =
           Room.databaseBuilder(
               context,
               AppDatabase::class.java,
               "app_database"
           ).build()

        @Provides
        fun provideUserDao(database: AppDatabase): UserDao = database.userDao()
    }
}
```

 - `@Module`：标记该类为一个 Dagger 模块，负责提供依赖。
 - `@InstallIn(SingletonComponent::class)`：指定该模块的安装范围为 `SingletonComponent`，意味着提供的依赖在整个应用生命周期内都是单例的。
 - `@Binds` 方法将 `OfflineUsersRepository` 实现类绑定到 `UsersRepository` 接口，使得在依赖注入时可以注入接口而不需要关心具体实现。  
 - `@Singleton`：确保实例在整个应用中是单例的。
 - `@Provides provideAppDatabase` 创建 `AppDatabase` 的实例。
 - `@Provides provideUserDao` 从 `AppDatabase` 实例中获取 `UserDao`

{% note default 不使用 interface 的 AppModule.kt %}
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase =
        Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        ).build()

    @Provides
    fun provideUserDao(database: AppDatabase): UserDao = database.userDao()

    @Provides
    @Singleton
    fun provideUsersRepository(userDao: UserDao): UsersRepository =
        UsersRepository(userDao)
}
```
{% endnote %}

可以编译一下，看看是否有错误。  

## 界面 

实现一个添加 User 的界面

### 创建 ViewModel

创建一个 `AddUserViewModel.kt` ，使用 `@HiltViewModel` 与 `@Inject` 提供 `usersRepository`   

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val usersRepository: UsersRepository
): ViewModel() {
    var uiState by mutableStateOf(UserUiState())

    suspend fun addUser(){
        usersRepository.insertUser(uiState.user)
        uiState = UserUiState()
    }

    fun updateUserUiState(user: User){
        uiState = UserUiState(
            user,
            isButtonEnable = isValidInput(user)
        )
    }

    private fun isValidInput(user: User): Boolean {
        return user.name.isNotEmpty() && user.bio.isNotEmpty()
    }
}

data class UserUiState (
    val user: User = User(
        name = "",
        bio = ""
    ),
    val isButtonEnable: Boolean = false
)
```

### 创建界面 Screen
创建一个 `AddUserScreen.kt` ，传递的是 **hiltViewModel()** 而不是 **viewModel()**

```kotlin
package com.example.simpleroom.ui 

import androidx.hilt.navigation.compose.hiltViewModel
import ...

@Composable
fun AddUserScreen(
    viewModel : UserViewModel = hiltViewModel()
){
    val coroutineScope = rememberCoroutineScope()

    Column{
        UserEntry(
            viewModel.uiState.user,
            viewModel::updateUserUiState
        )

        Button( onClick = { coroutineScope.launch {
                    viewModel.addUser()
                }
            },
            modifier = Modifier.fillMaxWidth().padding(8.dp),
            enabled = viewModel.uiState.isButtonEnable
        ) { Text("Add") }

    }
}

@Composable
fun UserEntry(
    user: User,
    onValueChange : (User) -> Unit,
    modifier: Modifier = Modifier
){
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = modifier
    ) {
        OutlinedTextField(
            value = user.name,
            onValueChange = { onValueChange(user.copy(name = it)) },
            label = { Text("Name:") },
            modifier = Modifier.fillMaxWidth().padding(8.dp),
            keyboardOptions = KeyboardOptions(imeAction = ImeAction.Next)
        )

        OutlinedTextField(
            value = user.bio,
            onValueChange = { onValueChange(user.copy(bio = it)) },
            label = { Text("Bio:") },
            modifier = Modifier.fillMaxWidth().padding(8.dp),
            keyboardOptions = KeyboardOptions(imeAction = ImeAction.Done)
        )
    }
}
```

修改 **MainActivity.kt** 调用 AddUserScreen()
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
//        enableEdgeToEdge()
        setContent {
            Surface(
                modifier = Modifier.fillMaxSize(),
                color = MaterialTheme.colorScheme.background
            ) {
                SimpleRoomTheme {
                    AddUserScreen()
            }  }

        }
    }
}
```

Build 运行后，可以打开 Android Studio 的 `View` > `Tool Windows` > `App Inspection` 查看数据库。  

### 使用 `Flow<List<User>>`

在 ViewModel 中定义
```kotlin
fun getAllUsers(): Flow<List<User>> = usersRepository.getAllUsers()
```

在 Screen 中获取 List<User>

```kotlin
val users by viewModel.getAllUsers().collectAsState(initial = emptyList())

LazyColumn {
    items(users){ user ->
        //....
    }
}
```
