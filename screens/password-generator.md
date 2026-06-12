# PasswordGeneratorScreen

## Visión General

Pantalla para generar contraseñas seguras con opciones configurables de longitud y tipos de caracteres.

**Archivo**: `presentation/ui/screens/PasswordGeneratorScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Generador de Contraseñas             │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  A7#k9L$mP2@nX5!q    [Copiar]   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Longitud: 16 ─────●─────────── 32      │
│                                         │
│  ☐ Mayúsculas (A-Z)                     │
│  ☑ Minúsculas (a-z)                     │
│  ☑ Números (0-9)                        │
│  ☑ Símbolos (!@#$...)                   │
│  ☐ Excluir ambiguos (0,O,1,l)           │
│                                         │
│  Fortaleza: ████████░░ Fuerte           │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │         GENERAR                 │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado de la UI

```kotlin
data class PasswordGeneratorState(
    val generatedPassword: String = "",
    val passwordLength: Int = 16,
    val useUppercase: Boolean = true,
    val useLowercase: Boolean = true,
    val useDigits: Boolean = true,
    val useSymbols: Boolean = true,
    val excludeAmbiguous: Boolean = false,
    val strengthScore: Int = 0,
    val strengthLabel: String = "Débil",
    val strengthColor: Long = 0xFFFF0000L,
    val showCopiedMessage: Boolean = false,
    val errorMessage: String? = null
)
```

## Eventos de Usuario

```kotlin
sealed class PasswordGeneratorEvent {
    data class OnLengthChanged(val length: Int) : PasswordGeneratorEvent()
    data class OnUseUppercaseChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseLowercaseChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseDigitsChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseSymbolsChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnExcludeAmbiguousChanged(val exclude: Boolean) : PasswordGeneratorEvent()
    data object OnGenerateClicked : PasswordGeneratorEvent()
    data object OnCopyClicked : PasswordGeneratorEvent()
    data object OnDismissCopiedMessage : PasswordGeneratorEvent()
}
```

## Cálculo de Fortaleza

```kotlin
fun calculateStrength(password: String): Int {
    var score = 0
    
    // Puntuación por longitud
    if (password.length >= 8) score += 20
    if (password.length >= 12) score += 20
    if (password.length >= 16) score += 20
    if (password.length >= 20) score += 10
    
    // Puntuación por variedad de caracteres
    if (password.any { it.isUpperCase() }) score += 15
    if (password.any { it.isLowerCase() }) score += 10
    if (password.any { it.isDigit() }) score += 15
    if (password.any { !it.isLetterOrDigit() }) score += 10
    
    return score.coerceIn(0, 100)
}

fun getStrengthLabel(score: Int): String = when {
    score < 40 -> "Débil"
    score < 70 -> "Media"
    score < 90 -> "Fuerte"
    else -> "Muy Fuerte"
}

fun getStrengthColor(score: Int): Long = when {
    score < 40 -> 0xFFFF0000L  // Rojo
    score < 70 -> 0xFFFFA500L  // Naranja
    score < 90 -> 0xFFFFFF00L  // Amarillo
    score < 100 -> 0xFF00FF00L // Verde
    else -> 0xFF008000L        // Verde oscuro
}
```

## Código del Componente

```kotlin
@Composable
fun PasswordGeneratorScreen(
    viewModel: PasswordGeneratorViewModel = koinViewModel(),
    onNavigateBack: () -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    val context = LocalContext.current
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Generador de Contraseñas") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.Default.ArrowBack, null)
                    }
                }
            )
        }
    ) { padding ->
        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(padding)
                .padding(16.dp)
        ) {
            // Password generada
            Card(
                modifier = Modifier.fillMaxWidth(),
                colors = CardDefaults.cardColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                )
            ) {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text(
                        text = state.generatedPassword,
                        style = MaterialTheme.typography.titleLarge,
                        fontFamily = FontFamily.Monospace
                    )
                    IconButton(onClick = {
                        // Copiar al portapapeles
                        val clipboard = context.getSystemService(
                            Context.CLIPBOARD_SERVICE
                        ) as ClipboardManager
                        clipboard.setPrimaryClip(
                            ClipData.newPlainText(
                                "Password",
                                state.generatedPassword
                            )
                        )
                        viewModel.onEvent(PasswordGeneratorEvent.OnCopyClicked)
                    }) {
                        Icon(Icons.Default.ContentCopy, "Copiar")
                    }
                }
            }
            
            Spacer(modifier = Modifier.height(24.dp))
            
            // Slider de longitud
            Text("Longitud: ${state.passwordLength}")
            Slider(
                value = state.passwordLength.toFloat(),
                onValueChange = { 
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnLengthChanged(it.toInt())
                    )
                },
                valueRange = 8f..32f,
                steps = 23
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // Opciones
            CheckboxWithLabel(
                checked = state.useUppercase,
                onCheckedChange = {
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnUseUppercaseChanged(it)
                    )
                },
                label = "Mayúsculas (A-Z)"
            )
            
            CheckboxWithLabel(
                checked = state.useLowercase,
                onCheckedChange = {
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnUseLowercaseChanged(it)
                    )
                },
                label = "Minúsculas (a-z)"
            )
            
            CheckboxWithLabel(
                checked = state.useDigits,
                onCheckedChange = {
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnUseDigitsChanged(it)
                    )
                },
                label = "Números (0-9)"
            )
            
            CheckboxWithLabel(
                checked = state.useSymbols,
                onCheckedChange = {
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnUseSymbolsChanged(it)
                    )
                },
                label = "Símbolos (!@#$...)"
            )
            
            CheckboxWithLabel(
                checked = state.excludeAmbiguous,
                onCheckedChange = {
                    viewModel.onEvent(
                        PasswordGeneratorEvent.OnExcludeAmbiguousChanged(it)
                    )
                },
                label = "Excluir ambiguos (0, O, 1, l, I)"
            )
            
            Spacer(modifier = Modifier.weight(1f))
            
            // Medidor de fortaleza
            Column(modifier = Modifier.fillMaxWidth()) {
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween
                ) {
                    Text("Fortaleza:")
                    Text(
                        text = state.strengthLabel,
                        color = Color(state.strengthColor)
                    )
                }
                
                LinearProgressIndicator(
                    progress = state.strengthScore / 100f,
                    modifier = Modifier.fillMaxWidth(),
                    color = Color(state.strengthColor)
                )
            }
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // Botón Generar
            Button(
                onClick = {
                    viewModel.onEvent(PasswordGeneratorEvent.OnGenerateClicked)
                },
                modifier = Modifier.fillMaxWidth()
            ) {
                Icon(Icons.Default.Refresh, null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("GENERAR")
            }
            
            // Mensaje de copiado
            if (state.showCopiedMessage) {
                Spacer(modifier = Modifier.height(8.dp))
                Text(
                    text = "✓ Copiado al portapapeles",
                    color = MaterialTheme.colorScheme.primary,
                    style = MaterialTheme.typography.bodySmall,
                    textAlign = TextAlign.Center,
                    modifier = Modifier.fillMaxWidth()
                )
            }
        }
    }
}
```

## Use Case: GeneratePassword

```kotlin
class GeneratePassword {
    suspend operator fun invoke(options: PasswordOptions): String {
        val charPool = buildCharPool(options)
        require(charPool.isNotEmpty()) {
            "Debe seleccionar al menos un tipo de carácter"
        }
        
        return (1..options.length)
            .map { SecureRandom().nextInt(charPool.size) }
            .map(charPool::get)
            .joinToString("")
    }
    
    private fun buildCharPool(options: PasswordOptions): String {
        return buildString {
            if (options.useUppercase) append('A'..'Z')
            if (options.useLowercase) append('a'..'z')
            if (options.useDigits) append('0'..'9')
            if (options.useSymbols) {
                append("!@#$%^&*()_+-=[]{}|;:,.<>?")
            }
            if (options.excludeAmbiguous) {
                // Eliminar caracteres ambiguos
                replace("0", "")
                replace("O", "")
                replace("1", "")
                replace("l", "")
                replace("I", "")
            }
        }
    }
}

data class PasswordOptions(
    val length: Int = 16,
    val useUppercase: Boolean = true,
    val useLowercase: Boolean = true,
    val useDigits: Boolean = true,
    val useSymbols: Boolean = true,
    val excludeAmbiguous: Boolean = false
)
```

## Navegación

```kotlin
composable(Screen.PasswordGenerator.route) {
    PasswordGeneratorScreen(
        viewModel = koinViewModel(),
        onNavigateBack = { navController.popBackStack() }
    )
}
```

## Referencias

- [PasswordGeneratorViewModel](../viewmodels/overview.md#passwordgeneratorviewmodel)
- [GeneratePassword Use Case](../domain/overview.md#generatepassword)
