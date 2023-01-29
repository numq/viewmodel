## ViewModel

ViewModel implementation focused on applying the *MVVM* architectural pattern and the *State Reduction* approach in
developing a *Kotlin Jetpack Compose Desktop* application

### Components

- CloseableCoroutineScope
- ViewModel
- StateViewModel

### Dependencies

```
val coroutinesVersion = "1.6.4"
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutinesVersion")
```

### Usage examples

- [Stash](https://github.com/numq/Stash) - Jetpack Compose Desktop

### Koin injection

```kotlin
val auth = module {
    factory { GetUserId(get()) }
    single { AuthViewModel(get()) }
}
```

### State managing

Use case example implementation:

```kotlin
class GetUserId<T> constructor(
    coroutineScope: CoroutineScope,
    arg: Unit,
    onException: (Exception) -> Unit,
    onResult: (T) -> Unit = {}
) : UseCase<Unit, T>()
```

Sealed class as state:

```kotlin
sealed class AuthState private constructor() {
    object Unauthenticated : AuthState()
    object Authenticating : AuthState()
    data class Authenticated(val userId: String? = null) : AuthState()
}

class AuthViewModel constructor(
    private val getUserId: GetUserId
) : StateViewModel<AuthState>(AuthState.Unauthenticated) {
    fun authenticate() = getUserId.invoke(viewModelScope, Unit, onException) { flow ->
        updateState { AuthState.Authenticating }
        viewModelScope.launch {
            flow.collect { id ->
                updateState {
                    id?.let { AuthState.Authenticated(id) } ?: AuthState.Unauthenticated
                }
            }
        }
    }
}
```

Data class as state:

```kotlin
data class AuthState(val userId: String? = null)

class AuthViewModel constructor(
    private val getUserId: (CoroutineScope, Unit, (Exception) -> Unit, (Flow<String?>) -> Unit) -> Unit
) : StateViewModel<AuthState>(AuthState()) {
    fun authenticate() = getUserId.invoke(viewModelScope, Unit, onException) { flow ->
        viewModelScope.launch {
            flow.collect { id ->
                updateState {
                    it.copy(userId = id)
                }
            }
        }
    }
}
```
