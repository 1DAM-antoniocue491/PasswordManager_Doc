# PasswordGeneratorWidget

## Visión General

Widget reutilizable para generar contraseñas dentro de formularios.

**Archivo**: `presentation/ui/components/PasswordGeneratorWidget.kt`

## Uso

```kotlin
@Composable
fun PasswordFormScreen(...) {
    var password by remember { mutableStateOf("") }
    var showGenerator by remember { mutableStateOf(false) }
    
    // Campo de contraseña
    OutlinedTextField(
        value = password,
        onValueChange = { password = it },
        label = { Text("Contraseña") },
        trailingIcon = {
            IconButton(onClick = { showGenerator = true }) {
                Icon(Icons.Default.Casino, "Generar")
            }
        }
    )
    
    if (showGenerator) {
        PasswordGeneratorWidget(
            onPasswordGenerated = { generated ->
                password = generated
                showGenerator = false
            },
            onDismiss = { showGenerator = false }
        )
    }
}
```

## API

```kotlin
@Composable
fun PasswordGeneratorWidget(
    onPasswordGenerated: (String) -> Unit,  // Callback con password generada
    onDismiss: () -> Unit,                  // Callback de dismiss
    initialLength: Int = 16,                // Longitud inicial
    modifier: Modifier = Modifier
)
```

## Referencias

- [PasswordGeneratorScreen](../screens/password-generator.md)
- [GeneratePassword Use Case](../domain/overview.md#generatepassword)
