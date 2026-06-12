# Uso de Koin en la Aplicación

Guía práctica para usar inyección de dependencias en diferentes contextos.

---

## Tipos de Binding

### single

Instancia única compartida en toda la aplicación:

```kotlin
single { KeystoreManager() }
single<AuthRepository> { AuthRepositoryImpl(...) }
```

**Cuándo usar**:
- Servicios sin estado
- Repositorios
- Use Cases
- Componentes de seguridad

---

### factory

Nueva instancia cada vez que se solicita:

```kotlin
factory { PasswordDeriver() }
```

**Cuándo usar**:
- Objetos ligeros sin estado compartido
- Cuando se necesita una instancia nueva

---

### viewModel

Instancia vinculada al lifecycle del ViewModel:

```kotlin
viewModel { PasswordListViewModel(get(), get()) }
```

**Cuándo usar**:
- Exclusivo para ViewModels
- Se limpia automáticamente

---

## Inyección en Compose

### koinViewModel

```kotlin
@Composable
fun PasswordListScreen(
    viewModel: PasswordListViewModel = koinViewModel(),
    onNavigateToDetail: (String) -> Unit,
    // ...
) {
    val state by viewModel.state.collectAsState()
    // ...
}
```

### koinInject

Para otras dependencias en Composables:

```kotlin
@Composable
fun SomeComponent(
    repository: PasswordRepository = koinInject()
) {
    // Usar repository
}
```

### getKoin

Para acceso manual fuera de Compose:

```kotlin
val koin = getKoin()
val repository = koin.get<PasswordRepository>()
```

---

## Inyección en Activities

```kotlin
class MainActivity : ComponentActivity() {

    private val autoLockManager: AutoLockManager by inject()

    override fun onCreate() {
        super.onCreate()
        
        // Usar autoLockManager
        enableEdgeToEdge()
        
        setContent {
            PasswordManagerTheme {
                NavGraph()
            }
        }
    }
}
```

---

## Inyección en Fragments

```kotlin
class BiometricFragment : Fragment() {

    private val biometricAuthenticator: BiometricAuthenticator by inject()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Usar biometricAuthenticator
    }
}
```

---

## Inyección en ViewModels

Dentro de un ViewModel, las dependencias se inyectan directamente en el constructor:

```kotlin
class PasswordListViewModel(
    private val getAllPasswords: GetAllPasswords,
    private val deletePasswordEntry: DeletePasswordEntry,
    private val toggleFavoriteEntry: ToggleFavoriteEntry
) : ViewModel() {
    // Las dependencias ya están disponibles
}
```

---

## Patrones Comunes

### Repository + Use Case + ViewModel

```kotlin
// Módulo
val appModule = module {
    // Repositorio
    single<PasswordRepository> { PasswordRepositoryImpl(get(), get(), get()) }
    
    // Use Case
    single { GetAllPasswords(get()) }
    
    // ViewModel
    viewModel { PasswordListViewModel(get(), get(), get()) }
}

// ViewModel
class PasswordListViewModel(
    private val getAllPasswords: GetAllPasswords,
    // ...
) : ViewModel() {
    init {
        loadPasswords()
    }
    
    private fun loadPasswords() {
        viewModelScope.launch {
            getAllPasswords().collect { entries ->
                // Actualizar estado
            }
        }
    }
}

// Composable
@Composable
fun PasswordListScreen(viewModel: PasswordListViewModel = koinViewModel()) {
    val state by viewModel.state.collectAsState()
    // UI
}
```

---

## Errores Comunes

### InstanceCreationException

```
org.koin.core.error.InstanceCreationException: Could not create instance
```

**Causa**: Una dependencia no está definida o hay un error en la inicialización.

**Solución**: Verificar que todas las dependencias estén registradas en el módulo.

---

### NoDefinitionFoundException

```
org.koin.core.error.NoDefinitionFoundException: No definition found for type X
```

**Causa**: Se intenta inyectar un tipo que no está registrado.

**Solución**: Añadir la definición al módulo:

```kotlin
single { MiClase() }
```

---

### DependencyCycleException

```
org.koin.core.error.DependencyCycleException
```

**Causa**: Ciclo circular de dependencias (A necesita B, B necesita A).

**Solución**: Reestructurar las dependencias para eliminar el ciclo.

---

### ScopeNotStartedException

```
org.koin.core.error.ScopeNotStartedException
```

**Causa**: Se intenta usar un scope que no ha sido iniciado.

**Solución**: Asegurar que el scope esté activo antes de usar.

---

## Testing con Koin

### Mocks en Tests

```kotlin
@Test
fun `should load passwords successfully`() = runTest {
    // Given
    val mockRepository = mockk<PasswordRepository>()
    val expectedEntries = listOf(/* ... */)
    coEvery { mockRepository.getAllEntries() } returns flowOf(expectedEntries)
    
    val viewModel = PasswordListViewModel(mockRepository, /* ... */)
    
    // When & Then
    viewModel.state.test {
        assertEquals(expectedEntries, awaitItem().entries)
    }
}
```

### Módulo de Testing

```kotlin
val testModule = module {
    single { mockk<PasswordRepository>() }
    single { mockk<AuthRepository>() }
    // ... más mocks
}

// En el test
startKoin {
    modules(testModule)
}
```
