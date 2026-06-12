# OnboardingScreen

## Visión General

Pantalla de introducción que guía al usuario en la configuración inicial de la aplicación.

**Archivo**: `presentation/ui/screens/OnboardingScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│                                         │
│            ┌───────────┐                │
│            │   🔐      │                │
│            │  Icono    │                │
│            └───────────┘                │
│                                         │
│     Bienvenido a Password Manager       │
│                                         │
│  Tu gestor de contraseñas seguro y      │
│  fácil de usar.                         │
│                                         │
│  ─────────────────────────────────      │
│                                         │
│  ¿Qué puedes hacer?                     │
│                                         │
│  ✓ Almacenar contraseñas cifradas       │
│  ✓ Generar contraseñas seguras          │
│  ✓ Organizar por categorías             │
│  ✓ Acceder con biometría                │
│  ✓ Hacer copias de seguridad            │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │         COMENZAR                │    │
│  └─────────────────────────────────┘    │
│                                         │
│            ○ ● ○                        │
│                                         │
└─────────────────────────────────────────┘
```

## Flujo de Pasos

```
┌──────────────────────────────────────────────────────────────┐
│ Paso 1: Bienvenida                                           │
│ "Bienvenido a Password Manager"                              │
│ Descripción de características                               │
│ Botón: "COMENZAR" → Paso 2                                   │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ Paso 2: Configurar Contraseña Maestra                        │
│ "Crea tu contraseña maestra"                                 │
│ Campo de contraseña con validación en tiempo real            │
│ Indicador de fortaleza                                       │
│ Botón: "CONTINUAR" → Paso 3                                  │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ Paso 3: Configurar Biometría                                 │
│ "¿Quieres usar biometría?"                                   │
│ Botón: "ACTIVAR HUELLA" → Configurar                         │
│ Botón: "SALTAR" → Finalizar                                  │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    Setup Completado
                    Navegar a Login
```

## Estado

```kotlin
data class OnboardingState(
    val currentStep: Int = 0,
    val totalSteps: Int = 3,
    val masterPassword: String = "",
    val confirmPassword: String = "",
    val passwordStrength: Int = 0,
    val isBiometricAvailable: Boolean = false,
    val enableBiometric: Boolean = true,
    val isLoading: Boolean = false,
    val error: String? = null
)
```

## Validaciones de Contraseña

```kotlin
object PasswordValidator {
    fun isValid(password: String): Boolean {
        return password.length >= 8 &&
               password.any { it.isUpperCase() } &&
               password.any { it.isDigit() } &&
               password.any { !it.isLetterOrDigit() }
    }
    
    fun getStrengthScore(password: String): Int {
        var score = 0
        if (password.length >= 8) score += 20
        if (password.length >= 12) score += 20
        if (password.length >= 16) score += 20
        if (password.any { it.isUpperCase() }) score += 15
        if (password.any { it.isLowerCase() }) score += 10
        if (password.any { it.isDigit() }) score += 15
        if (password.any { !it.isLetterOrDigit() }) score += 10
        return score.coerceIn(0, 100)
    }
}
```

## Use Case: SetupMasterPassword

```kotlin
class SetupMasterPassword(
    private val authRepository: AuthRepository,
    private val seedCategories: SeedPredefinedCategories
) {
    suspend operator fun invoke(password: CharArray): Result<Unit> {
        return try {
            // Validar fortaleza
            if (!PasswordValidator.isValid(password)) {
                return Result.failure(WeakPasswordException())
            }
            
            // Configurar contraseña maestra
            authRepository.setupMasterPassword(password)
            
            // Seed de categorías predefinidas
            seedCategories()
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

## Navegación

```kotlin
composable(Screen.Onboarding.route) {
    OnboardingScreen(
        viewModel = koinViewModel(),
        onSetupComplete = {
            navController.navigate(Screen.Login.route) {
                popUpTo(Screen.Onboarding.route) { inclusive = true }
            }
        }
    )
}
```

## Referencias

- [AuthViewModel](../viewmodels/overview.md#authviewmodel)
- [SetupMasterPassword Use Case](../domain/overview.md#setupmasterpassword)
