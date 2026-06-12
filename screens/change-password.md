# ChangePasswordScreen

## Visión General

Pantalla para cambiar la contraseña maestra del usuario. Requiere verificar la contraseña actual y re-cifra todas las entradas.

**Archivo**: `presentation/ui/screens/ChangePasswordScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Cambiar Contraseña Maestra           │
├─────────────────────────────────────────┤
│                                         │
│  ⚠️ Esta acción re-cifrará todas tus    │
│     contraseñas. Puede tardar unos      │
│     segundos.                           │
│                                         │
│  CONTRASEÑA ACTUAL                      │
│  ┌─────────────────────────────────┐    │
│  │ [••••••••••••••]        [👁️]    │    │
│  └─────────────────────────────────┘    │
│                                         │
│  NUEVA CONTRASEÑA                       │
│  ┌─────────────────────────────────┐    │
│  │ [••••••••••••••]        [👁️]    │    │
│  └─────────────────────────────────┘    │
│  Fortaleza: ████████░░ Fuerte           │
│                                         │
│  CONFIRMAR CONTRASEÑA                   │
│  ┌─────────────────────────────────┐    │
│  │ [••••••••••••••]        [👁️]    │    │
│  └─────────────────────────────────┘    │
│                                         │
│  REQUISITOS                             │
│  ✓ Mínimo 8 caracteres                  │
│  ✓ Al menos una mayúscula               │
│  ✓ Al menos un número                   │
│  ✓ Al menos un símbolo                  │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │      CAMBIAR CONTRASEÑA         │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class ChangePasswordState(
    val oldPassword: String = "",
    val newPassword: String = "",
    val confirmPassword: String = "",
    val isLoading: Boolean = false,
    val isReEncrypting: Boolean = false,
    val reEncryptProgress: Float = 0f,
    val error: String? = null,
    val successMessage: String? = null,
    val showOldPassword: Boolean = false,
    val showNewPassword: Boolean = false,
    val strengthScore: Int = 0
)
```

## Validaciones

| Campo | Regla | Mensaje de Error |
|-------|-------|------------------|
| oldPassword | No vacío | "La contraseña actual es requerida" |
| newPassword | Mínimo 8 caracteres | "Mínimo 8 caracteres" |
| newPassword | Mayúscula + número + símbolo | "Debe incluir mayúscula, número y símbolo" |
| confirmPassword | Igual a newPassword | "Las contraseñas no coinciden" |

## Flujo de Cambio

```
1. Usuario ingresa contraseña actual
         ↓
2. Verificar contraseña actual (AuthenticateUser)
         ↓
3. Validar nueva contraseña (PasswordValidator)
         ↓
4. Obtener todas las contraseñas
         ↓
5. Descifrar todas con clave antigua
         ↓
6. Configurar nueva contraseña maestra
         ↓
7. Re-cifrar todas con nueva clave
         ↓
8. Guardar en base de datos
         ↓
9. Cerrar sesión (requiere re-login)
```

## Use Case: ChangeMasterPassword

```kotlin
class ChangeMasterPassword(
    private val authRepository: AuthRepository,
    private val passwordRepository: PasswordRepository,
    private val cipherManager: CipherManager
) {
    suspend operator fun invoke(
        oldPassword: CharArray,
        newPassword: CharArray
    ): Result<Unit> {
        return try {
            // 1. Verificar contraseña actual
            val authResult = authRepository.authenticateUser(oldPassword)
            if (authResult.isFailure) {
                return Result.failure(InvalidCurrentPasswordException())
            }
            
            // 2. Validar nueva contraseña
            if (!PasswordValidator.isValid(newPassword)) {
                return Result.failure(WeakPasswordException())
            }
            
            // 3. Obtener todas las contraseñas
            val entries = passwordRepository.getAllEntriesList()
            
            // 4. Obtener clave actual
            val oldKey = authRepository.getMasterKey()!!
            
            // 5. Descifrar todas las entradas
            val decryptedEntries = entries.map { entry ->
                entry.copy(
                    password = dataCipher.decrypt(
                        entry.password.fromBase64String(), 
                        oldKey
                    ),
                    notes = entry.notes?.let {
                        dataCipher.decrypt(it.fromBase64String(), oldKey)
                    }
                )
            }
            
            // 6. Configurar nueva contraseña maestra
            authRepository.setupMasterPassword(newPassword)
            
            // 7. Re-cifrar con nueva clave
            val newKey = authRepository.getMasterKey()!!
            decryptedEntries.forEach { entry ->
                passwordRepository.updateEntry(
                    entry.copy(
                        password = dataCipher.encrypt(entry.password, newKey)
                            .toBase64String(),
                        notes = entry.notes?.let {
                            dataCipher.encrypt(it, newKey).toBase64String()
                        }
                    )
                )
            }
            
            // 8. Cerrar sesión
            authRepository.clearMasterKey()
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

## Navegación

```kotlin
composable(Screen.ChangePassword.route) {
    ChangePasswordScreen(
        viewModel = koinViewModel(),
        onNavigateBack = { navController.popBackStack() },
        onPasswordChanged = {
            // Forzar re-login
            navController.navigate(Screen.Login.route) {
                popUpTo(Screen.Home.route) { inclusive = true }
            }
        }
    )
}
```

## Referencias

- [ChangePasswordViewModel](../viewmodels/overview.md#changepasswordviewmodel)
- [ChangeMasterPassword Use Case](../domain/overview.md#changemasterpassword)
