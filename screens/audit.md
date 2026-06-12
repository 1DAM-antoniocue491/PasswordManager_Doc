# AuditScreen

## Visión General

Pantalla de auditoría que detecta contraseñas débiles y muestra recomendaciones de seguridad.

**Archivo**: `presentation/ui/screens/AuditScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Auditoría de Seguridad               │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐    │
│  │      Puntuación: 75/100         │    │
│  │    ████████░░░░ Buena           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  RESUMEN                                │
│  ┌─────────────────────────────────┐    │
│  │ Total contraseñas       45      │    │
│  │ Contraseñas débiles      8      │    │
│  │ Contraseñas reutilizadas 3      │    │
│  └─────────────────────────────────┘    │
│                                         │
│  CONTRASEÑAS DÉBILES                    │
│  ┌─────────────────────────────────┐    │
│  │ ⚠️  Google         < 8 caracteres│    │
│  │ ⚠️  Facebook       Sin símbolos  │    │
│  │ ⚠️  Amazon         Patrón común  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │     VER RECOMENDACIONES         │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class AuditState(
    val totalPasswords: Int = 0,
    val weakPasswords: Int = 0,
    val weakEntries: List<WeakPasswordEntry> = emptyList(),
    val securityScore: Int = 0,
    val isLoading: Boolean = false,
    val error: String? = null
)

data class WeakPasswordEntry(
    val id: String,
    val title: String,
    val reason: String
)
```

## Criterios de Debilidad

| Criterio | Descripción |
|----------|-------------|
| Longitud < 8 | Menos de 8 caracteres |
| Sin mayúsculas | No tiene letras mayúsculas |
| Sin números | No tiene dígitos numéricos |
| Sin símbolos | No tiene caracteres especiales |
| Patrón común | Contiene "123456", "password", "qwerty", etc. |

## Referencias

- [AuditViewModel](../viewmodels/overview.md#auditviewmodel)
- [AuditWeakPasswords Use Case](../domain/overview.md#auditweakpasswords)
