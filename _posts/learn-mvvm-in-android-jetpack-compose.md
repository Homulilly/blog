---
title: 学习笔记： Android Jetpack Compose 中的 MVVM 
date: 2024-10-06 09:36:56
tags:
 - Android
 - Compose
 - Jetpack
categories:
 - Note
---

## 介绍

在 [Android 之 Compose 开发基础 - 应用架构](https://developer.android.com/codelabs/basic-android-kotlin-compose-viewmodel-and-state?hl=zh-cn#0) 中介绍了 Jetpack Compose 常用的 **MVVM（Model-View-ViewModel）架构** 。

在源文件中一般分为 `Screen`、`UiState`、`ViewModel` 三个文件。

对应如下
- `Screen` - **View（视图）**
    - 由 `Screen` 中的 `@Composable` 函数组成
    - 负责界面的渲染和用户交互
    - 通过观察 ViewModel 中的状态（ `uiState` ）来观察界面。
- `ViewModel` - **ViewModel（视图模型）**
    - 持有并管理界面状态 `uiState`
    - 处理用户逻辑和用户事件
    - 使用 `StateFlow` 、 `LiveData` 或其他可观察的数据类型，将状态暴漏给视图 
- `UiState` - **Model（模型）** 
    - 负责数据管理和业务逻辑，如数据模型、仓库（Repository）、网络请求等。
    - 数据模型封装界面所需的状态数据，通常为不可变的数据类。
    - `ViewModel` 与 `Model` 交互，获取或更新数据

<!--more-->

## 特点
 - **单向数据流**：数据从 `ViewModel` 流向 `Screen`，用户交互从 `Screen` 通过事件回传给 `ViewModel`。降低了复杂性。
 - **状态不可变**：使用不可变的 `UiState`，使状态变化可预测，减少了调试困难。
 - **关注点分离**：将界面展示、状态管理和业务逻辑分离，提高了代码的可读性和可维护性。  
```
用户交互
    ↓
View（Screen）
    ↓           ↑
    事件       观察
    ↓           ↑
ViewModel（持有 uiState）
    ↓
与 Model 交互
```

## 使用
### 设置依赖

`build.gradle.kts(Module :app)`
```kotlin
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.1")
```

或是使用 Version Catalog
`libs.versions.toml`
```toml
[versions]
lifecycleViewModelCompose="2.8.6"

[libraries]
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleViewModelCompose" }
``` 

`build.gradle.kts(Module :app)`
```kotlin
implementation(libs.androidx.lifecycle.viewmodel.compose)
```


### UiState
`UiState` 是一个数据类，封装了界面展示所需的所有状态信息。它通常是不可变的，以确保状态的一致性和可预测性。

创建数据类 `GameUiState.kt`

```kotlin
data class GameUiState(
    val currentScrambledWord: String = "",
    val isGuessedWordWrong: Boolean = false,
    val score: Int = 0,
    val currentWordCount: Int = 1,
    val isGameOver: Boolean = false
)
```

### ViewModel
`ViewModel` 负责处理业务逻辑和状态管理。它持有 `UiState`，并通过 `StateFlow`、`LiveData` 或 `MutableState` 将状态暴露给界面层。

创建 `GameViewModel.kt`

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.lifecycle.viewmodel.compose.viewModel
// ...

@Composable
class GameViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(GameUiState())
    val uiState: StateFlow<GameUiState> = _uiState.asStateFlow()

    fun updateUiState(){
        // 其他代码
        // 更新状态
        _uiState.update { currentState ->
            currentState.copy(
                //...
            )
        }
    }
}
```

`_uiState`：私有的可变状态。
`uiState`：公开的只读状态，供界面层观察。
`updateUiState`：处理来自界面层的事件，更新 `uiState`。

### Screen 
`Screen` 是一个或多个 `@Composable` 函数，调用 `ViewModel` 负责根据传入的 `uiState` 渲染界面，并通过回调或事件将用户交互传递给 `ViewModel`。 

创建 `GameScreen.kt`
```kotlin
@Composable
fun GameScreen(
    gameViewModel: GameViewModel = viewModel()
){
    val gameUiState by gameViewModel.uiState.collectAsState() 
    // 等同于
    // val gameUiState = gameViewModel.uiState.collectAsState().value

    //....
    Button(
        onClick = { gameViewModel.updateUiState() } 
        // onClick = gameViewModel::updateUiState
    ){
        Text("Button")
    }
}
```

## 参考示例 By ChatGPT

```kotlin
// 定义事件
sealed class LoginEvent {
    data class OnUsernameChanged(val username: String) : LoginEvent()
    data class OnPasswordChanged(val password: String) : LoginEvent()
    object OnLoginClicked : LoginEvent()
}

// 定义界面状态
data class LoginUiState(
    val username: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)

// ViewModel 实现
class LoginViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun onEvent(event: LoginEvent) {
        when (event) {
            is LoginEvent.OnUsernameChanged -> {
                _uiState.update { it.copy(username = event.username) }
            }
            is LoginEvent.OnPasswordChanged -> {
                _uiState.update { it.copy(password = event.password) }
            }
            is LoginEvent.OnLoginClicked -> {
                login()
            }
        }
    }

    private fun login() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, errorMessage = null) }
            // 模拟网络请求
            delay(2000)
            val success = _uiState.value.username == "user" && _uiState.value.password == "pass"
            if (success) {
                _uiState.update { it.copy(isLoading = false) }
                // 处理登录成功
            } else {
                _uiState.update { it.copy(isLoading = false, errorMessage = "登录失败") }
            }
        }
    }
}

// 界面层
@Composable
fun LoginScreen(uiState: LoginUiState, onEvent: (LoginEvent) -> Unit) {
    Column {
        TextField(
            value = uiState.username,
            onValueChange = { onEvent(LoginEvent.OnUsernameChanged(it)) },
            label = { Text("用户名") }
        )
        TextField(
            value = uiState.password,
            onValueChange = { onEvent(LoginEvent.OnPasswordChanged(it)) },
            label = { Text("密码") },
            visualTransformation = PasswordVisualTransformation()
        )
        if (uiState.errorMessage != null) {
            Text(text = uiState.errorMessage, color = Color.Red)
        }
        Button(
            onClick = { onEvent(LoginEvent.OnLoginClicked) },
            enabled = !uiState.isLoading
        ) {
            if (uiState.isLoading) {
                CircularProgressIndicator()
            } else {
                Text("登录")
            }
        }
    }
}

// 将所有部分连接
@Composable
fun LoginScreenWrapper(viewModel: LoginViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    LoginScreen(
        uiState = uiState,
        onEvent = viewModel::onEvent
    )
}
```

