# CLAUDE.md - Moodle GIFT Viewer

## Descripción del Proyecto

Visualizador y editor de preguntas en formato GIFT para Moodle. Permite previsualizar, editar y exportar archivos GIFT con soporte completo para contenido multilingüe (mlang).

**URL de producción:** https://jlmirallesb.github.io/Moodle_GIFT_viewer/

## Stack Tecnológico

- **HTML5 + CSS3 + JavaScript vanilla (ES6+)**
- **Sin dependencias externas** (no npm, no frameworks, no CDN)
- **Single Page Application** - Todo el código está en `index.html` (3,492 líneas)

## Estructura del Archivo index.html

| Sección | Líneas | Contenido |
|---------|--------|-----------|
| CSS | 1-1180 | Estilos, tema oscuro, responsive |
| HTML | 1182-1257 | Estructura de la página |
| JavaScript | 1259-3492 | Lógica de la aplicación |

### Organización del JavaScript

- **Líneas 1261-1559**: Traducciones (7 idiomas: es, ca, eu, gl, en, fr, de)
- **Líneas 1552-1558**: Variables globales de estado
- **Líneas 1560-1650**: Gestión de idioma UI y tema
- **Líneas 1670-1756**: Selector de idioma de contenido
- **Líneas 1774-1875**: Funciones de edición básica
- **Líneas 1975-2219**: Creación de nuevas preguntas
- **Líneas 2222-2327**: Guardado de ediciones (saveEdit)
- **Líneas 2367-2490**: Generación y exportación GIFT
- **Líneas 2491-2859**: **CORE: Parsing de formato GIFT**
- **Líneas 3008-3412**: Renderizado de preguntas
- **Líneas 3414-3492**: Utilidades e inicialización

## Variables Globales de Estado

```javascript
let originalParsedQuestions = [];      // Preguntas parseadas originales
let editedQuestions = {};              // Ediciones por pregunta e idioma
let questionLanguageOverrides = {};    // Override de idioma por pregunta
let currentLanguage = 'es';            // Idioma de la interfaz
let currentTheme = 'system';           // Tema actual (light/dark/system)
```

## Tipos de Preguntas Soportados

| Tipo | Identificador | Formato GIFT |
|------|---------------|--------------|
| Opción Múltiple | `multipleChoice` | `~opcion =correcta` |
| Respuesta Múltiple | `multipleResponse` | `~%50%opcion ~%50%opcion` |
| Verdadero/Falso | `trueFalse` | `{T}` o `{F}` |
| Respuesta Corta | `shortAnswer` | `{=resp1 =resp2}` |
| Emparejamiento | `matching` | `{=pregunta -> respuesta}` |
| Numérica | `numerical` | `{#valor:margen}` |
| Ensayo | `essay` | `{}` |
| Palabra Faltante | `missingWord` | `texto {=resp} más texto` |

## Funciones Principales

### Parsing
- `parseGIFT(giftCode)` - Entrada principal, divide en bloques
- `parseGIFTFormat(block)` - Extrae título, texto y respuestas
- `parseQuestion(block)` - Detecta tipo y parsea respuestas
- `parseMultipleChoiceAnswers()`, `parseShortAnswers()`, etc.

### Renderizado
- `displayQuestions(questions)` - Renderiza lista de preguntas
- `renderAnswers(question, qIndex)` - Delega según tipo
- `renderMultipleChoice()`, `renderTrueFalse()`, etc.

### Edición
- `startEdit(qIndex, field, lang)` - Inicia modo edición
- `saveEdit(qIndex, field, lang)` - Guarda cambios
- `cancelEdit(qIndex, field, lang)` - Cancela edición

### Multilingüe (mlang)
- `processMlangText(text, lang)` - Extrae texto del idioma especificado
- `detectContentLanguages(questions)` - Detecta idiomas en el contenido
- `reconstructMlangText(original, edited, lang)` - Reconstruye con ediciones

### Exportación
- `generateGIFTCode()` - Genera código GIFT desde estado actual
- `copyGiftCode()` - Copia al portapapeles
- `downloadGiftFile()` - Descarga código editado como archivo .gift
- `downloadOriginalGiftFile()` - Descarga código original (sin editar) como archivo .gift

## Formato mlang de Moodle

El proyecto soporta el filtro multilingüe de Moodle:

```gift
::Título::
{mlang es}¿Pregunta en español?{mlang}
{mlang en}Question in English?{mlang}
{
=respuesta
}
```

**Nota:** En GIFT, las llaves se escapan: `\{mlang es\}...\{mlang\}`

## Patrones de Código

### Estructura de edición
```javascript
editedQuestions[qIndex] = {
    langs: new Set(['es', 'en']),  // Idiomas editados
    edits: {
        es: { title: "...", text: "...", answers: [...] },
        en: { title: "...", text: "...", answers: [...] }
    }
}
```

### Renderizado con HTML dinámico
```javascript
function renderTipoX(question, qIndex) {
    const t = translations[currentLanguage];
    const lang = questionLanguageOverrides[qIndex] || currentContentLanguage || 'es';
    return `<div class="...">
        ${processMlangText(question.text, lang)}
    </div>`;
}
```

### Actualización de UI
Después de cambios, llamar:
```javascript
filterAndDisplayQuestions();  // Re-renderiza preguntas
updateExportSection();        // Actualiza preview GIFT
```

## Comandos de Desarrollo

```bash
# Abrir en navegador (desarrollo)
open index.html

# Servir localmente
python -m http.server 8000

# Ver en http://localhost:8000
```

## Convenciones

- **Nombrado**: camelCase para funciones y variables
- **IDs HTML**: Mezcla de camelCase (`giftInput`) y kebab-case (`help-modal`)
- **Seguridad**: Usar `escapeHtml()` para prevenir XSS
- **Traducciones**: Acceder vía `translations[currentLanguage].clave`

## Temas

El proyecto soporta 3 temas: `light`, `dark`, `system`

```javascript
setTheme('dark');     // Activa tema oscuro
applyTheme();         // Aplica tema según configuración
```

Los estilos de tema oscuro usan el selector `[data-theme="dark"]`.

## Consideraciones al Modificar

1. **Todo está en un archivo**: Los cambios afectan todo el proyecto
2. **Regex sensibles**: El parsing depende de expresiones regulares complejas
3. **Estado compartido**: Las variables globales mantienen la sincronización
4. **Escapado de caracteres**: GIFT usa `\{`, `\}`, `\:`, `\~`, `\=`, `\#`
5. **Compatibilidad mlang**: Mantener soporte para ambos formatos (`{mlang}` y `\{mlang\}`)

## Testing Manual

1. Cargar ejemplo con "Cargar Ejemplo"
2. Probar edición de cada tipo de pregunta
3. Verificar que el código GIFT generado sea válido
4. Probar cambio de idioma de contenido
5. Probar modo oscuro
6. Verificar exportación (copiar/descargar código editado)
7. Verificar descarga de código original (botón aparece tras visualizar)

## Licencia

MIT License - José Luis Miralles Bono
