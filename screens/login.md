# LoginScreen

## Visión General

Pantalla de autenticación inicial que permite al usuario acceder a la aplicación mediante su contraseña maestra o autenticación biométrica.

**Archivo**: `presentation/ui/screens/LoginScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│            Logo / Icono                 │
│                                         │
│     ┌─────────────────────────┐         │
│     │   Contraseña maestra    │         │
│     │   [••••••••••••]        │         │
│     └─────────────────────────┘         │
│                                         │
│     ┌─────────────────────────┐         │
│     │      INICIAR SESIÓN     │         │
│     └─────────────────────────┘         │
│                                         │
│     ┌─────────────────────────┐         │
│     │  �ingerprint  Biometría │         │
│     └─────────────────────────┘         │
│                                         │
│           ¿Olvidaste tu contraseña?     │
└─────────────────────────────────────────┘
```

## Estado de la UI

```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false,
    val showBiometricPrompt: Boolean = false,
    val isSetupComplete: Boolean = false,
    val isAuthenticated: Boolean = false
)
```

## Eventos de Usuario

```kotlin
sealed class LoginEvent {
    data class OnPasswordChanged(val password: String) : LoginEvent()
    data object OnLoginClicked : LoginEvent()
    data object OnBiometricClicked : LoginEvent()
    data object OnBiometricSuccess : LoginEvent()
    data object OnBiometricError : LoginEvent()
    data object OnForgotPasswordClicked : LoginEvent()
}
```

## Flujo de Autenticación

### 1. Primer Uso (Sin Setup)

```
LoginScreen → OnboardingScreen → SetupMasterPassword
```

### 2. Usuario con Setup Completado

```
LoginScreen → Ingresar password → AuthenticateUser → HomeScreen
```

### 3. Autenticación Biométrica

```
LoginScreen → Click en biometría → BiometricPrompt 
           → AuthRepository.authenticateUser() → HomeScreen
```

## Código del Componente

```kotlin
@Composable
fun LoginScreen(
    viewModel: AuthViewModel = koinViewModel(),
    onLoginSuccess: () -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    var passwordText by remember { mutableStateOf("") }
    
    LaunchedEffect(state.isAuthenticated) {
        if (state.isAuthenticated) {
            onLoginSuccess()
        }
    }
    
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        // Logo
        Icon(
            imageVector = Icons.Default.Lock,
            contentDescription = null,
            modifier = Modifier.size(80.dp),
            tint = MaterialTheme.colorScheme.primary
        )
        
        Spacer(modifier = Modifier.height(32.dp))
        
        // Título
        Text(
            text = "Password Manager",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Spacer(modifier = Modifier.height(48.dp))
        
        // Campo de contraseña
        OutlinedTextField(
            value = passwordText,
            onValueChange = { passwordText = it },
            label = { Text("Contraseña maestra") },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Password,
                imeAction = ImeAction.Done
            ),
            singleLine = true,
            modifier = Modifier.fillMaxWidth(),
            isError = state.error != null
        )
        
        // Mensaje de error
        state.error?.let { error ->
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = error,
                color = MaterialTheme.colorScheme.error,
                style = MaterialTheme.typography.bodySmall
            )
        }
        
        Spacer(modifier = Modifier.height(24.dp))
        
        // Botón de inicio de sesión
        Button(
            onClick = { viewModel.onLoginClick(passwordText) },
            enabled = !state.isLoading && passwordText.isNotBlank(),
            modifier = Modifier.fillMaxWidth()
        ) {
            if (state.isLoading) {
                CircularProgressIndicator(
                    modifier = Modifier.size(24.dp),
                    color = MaterialTheme.colorScheme.onPrimary
                )
            } else {
                Text("Iniciar Sesión")
            }
        }
        
        // Botón biométrico (si está disponible)
        if (state.isBiometricAvailable) {
            Spacer(modifier = Modifier.height(16.dp))
            
            OutlinedButton(
                onClick = { viewModel.onBiometricAuthClick() },
                enabled = !state.isLoading,
                modifier = Modifier.fillMaxWidth()
            ) {
                Icon(
                    imageVector = Icons.Default.Fingerprint,
                    contentDescription = null,
                    modifier = Modifier.size(20.dp)
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text("Usar biometría")
            }
        }
    }
}
```

## Validaciones

| Campo | Regla | Mensaje de Error |
|-------|-------|------------------|
| Password | No vacío | "La contraseña es requerida" |
| Password | Mínimo 1 carácter | "Contraseña inválida" |

## Integración con ViewModel

```kotlin
class AuthViewModel(
    private val setupMasterPassword: SetupMasterPassword,
    private val authenticateUser: AuthenticateUser,
    private val authRepository: AuthRepository,
    private val autoLockManager: AutoLockManager,
    private val application: Application
) : ViewModel() {
    
    fun onLoginClick(password: String) {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)
            
            val result = authenticateUser(password.toCharArray())
            
            result.onSuccess { masterKey ->
                authRepository.setMasterKey(masterKey)
                autoLockManager.unlock()
                _state.value = _state.value.copy(
                    isLoading = false,
                    isAuthenticated = true
                )
            }.onFailure { error ->
                _state.value = _state.value.copy(
                    isLoading = false,
                    error = error.message ?: "Error de autenticación"
                )
            }
        }
    }
    
    fun onBiometricAuthClick() {
        // Solicita biometría y luego llama a authenticateUser
    }
}
```

## Navegación

```kotlin
composable(Screen.Login.route) {
    LoginScreen(
        viewModel = viewModel,
        onLoginSuccess = {
            navController.navigate(Screen.Home.route) {
                popUpTo(Screen.Login.route) { inclusive = true }
            }
        }
    )
}
```

## Referencias

- [AuthViewModel](../viewmodels/overview.md#authviewmodel)
- [BiometricAuthenticator](../security/overview.md#biometricauthenticator)
- [AuthenticateUser](../domain/overview.md#authenticateuser)
