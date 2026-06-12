# PasswordDetailScreen

## Visión General

Pantalla que muestra el detalle completo de una contraseña guardada, con opciones de copiado y edición.

**Archivo**: `presentation/ui/screens/PasswordDetailScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Detalle          [✏️ Editar] [🗑️]   │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐    │
│  │           Google                │    │
│  │         🔐 Favorito             │    │
│  └─────────────────────────────────┘    │
│                                         │
│  USUARIO                                │
│  ┌─────────────────────────────────┐    │
│  │ user@gmail.com          [📋]    │    │
│  └─────────────────────────────────┘    │
│                                         │
│  CONTRASEÑA                             │
│  ┌─────────────────────────────────┐    │
│  │ ••••••••••••••••        [👁️]    │    │
│  │ [📋 Copiar]   [🎲 Generar]      │    │
│  └─────────────────────────────────┘    │
│                                         │
│  URL                                    │
│  ┌─────────────────────────────────┐    │
│  │ https://google.com       [🔗]   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  CATEGORÍA                              │
│  ┌─────────────────────────────────┐    │
│  │ 📁 Redes Sociales               │    │
│  └─────────────────────────────────┘    │
│                                         │
│  NOTAS                                  │
│  ┌─────────────────────────────────┐    │
│  │ Cuenta personal de Gmail        │    │
│  │ Creada el 01/01/2024            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Última actualización: 10/06/2026       │
│  Creado: 01/01/2024                     │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class PasswordDetailState(
    val entry: PasswordEntry? = null,
    val showPassword: Boolean = false,
    val isLoading: Boolean = true,
    val error: String? = null,
    val showDeleteConfirmation: Boolean = false,
    val copiedField: String? = null  // "username", "password", "url", null
)
```

## Funcionalidades

### Copiar al Portapapeles

```kotlin
fun copyUsername(): ClipboardResult {
    val username = _state.value.entry?.username
    if (username.isNullOrBlank()) {
        return ClipboardResult(false, "No hay usuario para copiar")
    }
    
    val clip = ClipData.newPlainText("Password Manager", username)
    clipboardManager.setPrimaryClip(clip)
    return ClipboardResult(true, "Usuario copiado")
}

fun copyPassword(): ClipboardResult {
    val password = _state.value.entry?.password
    if (password.isNullOrBlank()) {
        return ClipboardResult(false, "No hay contraseña para copiar")
    }
    
    val clip = ClipData.newPlainText("Password Manager", password)
    clipboardManager.setPrimaryClip(clip)
    return ClipboardResult(true, "Contraseña copiada")
}
```

### Toggle Visibilidad de Contraseña

```kotlin
fun togglePasswordVisibility() {
    _state.value = _state.value.copy(
        showPassword = !_state.value.showPassword
    )
}
```

### Navegación a Edición

```kotlin
onNavigateToEdit = { entryId ->
    navController.navigate(Screen.PasswordEdit.createRoute(entryId))
}
```

### Confirmación de Eliminación

```kotlin
AlertDialog(
    onDismissRequest = { viewModel.dismissDeleteConfirmation() },
    title = { Text("Eliminar contraseña") },
    text = { Text("¿Estás seguro de que quieres eliminar esta contraseña? Esta acción no se puede deshacer.") },
    confirmButton = {
        TextButton(
            onClick = {
                viewModel.deleteEntry()
                viewModel.dismissDeleteConfirmation()
            }
        ) {
            Text("Eliminar", color = MaterialTheme.colorScheme.error)
        }
    },
    dismissButton = {
        TextButton(onClick = { viewModel.dismissDeleteConfirmation() }) {
            Text("Cancelar")
        }
    }
)
```

## Código del Componente

```kotlin
@Composable
fun PasswordDetailScreen(
    entryId: String,
    viewModel: PasswordDetailViewModel = koinViewModel(),
    onNavigateBack: () -> Unit,
    onNavigateToEdit: (String) -> Unit,
    onRequestDelete: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    val context = LocalContext.current
    
    LaunchedEffect(entryId) {
        viewModel.loadEntry(entryId)
    }
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Detalle") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.Default.ArrowBack, null)
                    }
                },
                actions = {
                    IconButton(onClick = { onNavigateToEdit(entryId) }) {
                        Icon(Icons.Default.Edit, "Editar")
                    }
                    IconButton(onClick = { 
                        state.entry?.let { viewModel.requestDelete(it) }
                    }) {
                        Icon(Icons.Default.Delete, "Eliminar", 
                             tint = MaterialTheme.colorScheme.error)
                    }
                }
            )
        }
    ) { padding ->
        when {
            state.isLoading -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
            state.error != null -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Text(state.error!!, color = MaterialTheme.colorScheme.error)
                }
            }
            state.entry != null -> {
                val entry = state.entry!!
                Column(
                    modifier = modifier
                        .fillMaxSize()
                        .padding(padding)
                        .verticalScroll(rememberScrollState())
                        .padding(16.dp)
                ) {
                    // Título
                    Card(
                        modifier = Modifier.fillMaxWidth(),
                        colors = CardDefaults.cardColors(
                            containerColor = MaterialTheme.colorScheme.primaryContainer
                        )
                    ) {
                        Column(
                            modifier = Modifier.padding(16.dp),
                            horizontalAlignment = Alignment.CenterHorizontally
                        ) {
                            Text(
                                text = entry.title,
                                style = MaterialTheme.typography.headlineSmall,
                                fontWeight = FontWeight.Bold
                            )
                            if (entry.isFavorite) {
                                Row(
                                    verticalAlignment = Alignment.CenterVertically,
                                    modifier = Modifier.padding(top = 8.dp)
                                ) {
                                    Icon(
                                        Icons.Default.Star,
                                        contentDescription = "Favorito",
                                        tint = Color(0xFFFFD700),
                                        modifier = Modifier.size(20.dp)
                                    )
                                    Spacer(modifier = Modifier.width(4.dp))
                                    Text(
                                        "Favorito",
                                        style = MaterialTheme.typography.labelSmall
                                    )
                                }
                            }
                        }
                    }
                    
                    Spacer(modifier = Modifier.height(24.dp))
                    
                    // Campo: Usuario
                    DetailField(
                        label = "USUARIO",
                        value = entry.username,
                        onCopy = { viewModel.copyUsername() }
                    )
                    
                    Spacer(modifier = Modifier.height(16.dp))
                    
                    // Campo: Contraseña
                    DetailField(
                        label = "CONTRASEÑA",
                        value = if (state.showPassword) entry.password else "•".repeat(entry.password.length),
                        onCopy = { viewModel.copyPassword() },
                        trailingIcon = {
                            IconButton(onClick = { viewModel.togglePasswordVisibility() }) {
                                Icon(
                                    if (state.showPassword) Icons.Default.VisibilityOff 
                                    else Icons.Default.Visibility,
                                    "Visibilidad"
                                )
                            }
                        }
                    )
                    
                    Spacer(modifier = Modifier.height(16.dp))
                    
                    // Campo: URL
                    entry.url?.let { url ->
                        DetailField(
                            label = "URL",
                            value = url,
                            onCopy = { 
                                val clip = ClipData.newPlainText("URL", url)
                                context.getSystemService(ClipboardManager::class.java)
                                    .setPrimaryClip(clip)
                            },
                            trailingIcon = {
                                IconButton(onClick = { 
                                    Intent(Intent.ACTION_VIEW, Uri.parse(url)).also {
                                        context.startActivity(it)
                                    }
                                }) {
                                    Icon(Icons.Default.OpenInNew, "Abrir")
                                }
                            }
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                    }
                    
                    // Campo: Categoría
                    DetailField(
                        label = "CATEGORÍA",
                        value = entry.categoryName ?: "Sin categoría"
                    )
                    
                    Spacer(modifier = Modifier.height(16.dp))
                    
                    // Campo: Notas
                    entry.notes?.let { notes ->
                        DetailField(
                            label = "NOTAS",
                            value = notes,
                            isMultiline = true
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                    }
                    
                    // Timestamps
                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        Text(
                            text = "Creado: ${formatDate(entry.createdAt)}",
                            style = MaterialTheme.typography.bodySmall,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                        Text(
                            text = "Actualizado: ${formatDate(entry.updatedAt)}",
                            style = MaterialTheme.typography.bodySmall,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
            }
        }
        
        // Diálogo de confirmación de eliminación
        if (state.showDeleteConfirmation) {
            DeleteConfirmationDialog(
                onDismiss = { viewModel.dismissDeleteConfirmation() },
                onConfirm = { 
                    state.entry?.let { 
                        viewModel.deleteEntry(it)
                        onRequestDelete(it.id)
                    }
                }
            )
        }
    }
}

@Composable
private fun DetailField(
    label: String,
    value: String,
    isMultiline: Boolean = false,
    onCopy: (() -> Unit)? = null,
    trailingIcon: @Composable (() -> Unit)? = null
) {
    Column {
        Text(
            text = label,
            style = MaterialTheme.typography.labelMedium,
            color = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.height(4.dp))
        OutlinedTextField(
            value = value,
            onValueChange = {},
            readOnly = true,
            singleLine = !isMultiline,
            maxLines = if (isMultiline) 5 else 1,
            modifier = Modifier.fillMaxWidth(),
            trailingIcon = {
                Row {
                    trailingIcon?.invoke()
                    onCopy?.let {
                        IconButton(onClick = it) {
                            Icon(Icons.Default.ContentCopy, "Copiar")
                        }
                    }
                }
            }
        )
    }
}
```

## Referencias

- [PasswordDetailViewModel](../viewmodels/overview.md#passworddetailviewmodel)
- [GetPasswordById Use Case](../domain/overview.md#getpasswordbyid)
