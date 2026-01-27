# 🔌 Sistema de Plugins - edAItorial Checkers

## ✅ Arquitectura de Plugins Implementada

El módulo **edAItorial** ahora usa un sistema de **plugins** para los checkers, haciéndolo modular, extensible y mantenible.

---

## 🎯 Plugins Implementados

### 1. **SEO Checker** (`seo`)
**ID:** `seo`  
**Categoría:** SEO  
**Peso:** 5

**Checks:**
- Title length optimization (min 10, max 70)
- Duplicate titles detection
- Meta description analysis
- Content length (min 300 words)
- Multiple H1 tags detection
- Text-to-HTML ratio
- Keywords in content
- URL alias validation

---

### 2. **Broken Links Checker** (`broken_links`)
**ID:** `broken_links`  
**Categoría:** SEO  
**Peso:** 10

**Checks:**

**HTML Links:**
- Empty links (`href=""`)
- Hash only links (`href="#"`)
- Broken internal links (`/node/999`)
- Poor anchor text ("click here", "read more")
- External links without rel="noopener"

**Link Fields:**
- Broken entity links (`entity:node/999`)
- Broken internal links (`internal:/node/999`)
- Empty URIs
- Missing link titles
- Poor link titles

---

### 3. **Typos Checker** (`typos`) ⭐ NUEVO
**ID:** `typos`  
**Categoría:** Content  
**Peso:** 20

**Checks:**
- Common typos detection (50+ common misspellings)
- Typos in title
- Typos in body content
- Repeated words detection ("the the")
- Severity based on typo count:
  - **High:** > 10 typos
  - **Medium:** 5-10 typos  
  - **Low:** 1-4 typos

**Typos Dictionary Includes:**
```
teh → the
recieve → receive
definately → definitely
goverment → government
alot → a lot
beleive → believe
becuase → because
tommorow → tomorrow
wich → which
thier → their
... (50+ total)
```

---

### 4. **Suggestions Checker** (`suggestions`) ⭐ NUEVO
**ID:** `suggestions`  
**Categoría:** Content  
**Peso:** 30

**Provides Suggestions For:**
- Content structure (headings for long content)
- Lists usage (bullet points)
- Visual content (images)
- Readability (sentence length)
- Paragraph structure
- Active vs passive voice
- Call-to-action presence
- External links for credibility
- Power words in title ("How", "Guide", "Best")
- Numbers in title ("10 Tips")

---

## 📁 Estructura de Archivos

```
web/modules/custom/edaitorial/
├── src/
│   ├── Annotation/
│   │   └── EdaitorialChecker.php           ← Anotación para plugins
│   ├── Plugin/
│   │   ├── EdaitorialCheckerInterface.php  ← Interface base
│   │   ├── EdaitorialCheckerBase.php       ← Clase base
│   │   └── EdaitorialChecker/
│   │       ├── SeoChecker.php              ← Plugin SEO
│   │       ├── BrokenLinksChecker.php      ← Plugin Broken Links
│   │       ├── TyposChecker.php            ← Plugin Typos ⭐
│   │       └── SuggestionsChecker.php      ← Plugin Suggestions ⭐
│   ├── EdaitorialCheckerManager.php        ← Plugin Manager
│   └── Service/
│       └── MetricsCollector.php            ← Usa el plugin manager
└── edaitorial.services.yml                 ← Registra el plugin manager
```

---

## 🔧 Cómo Funciona

### 1. Plugin Manager

El `EdaitorialCheckerManager` gestiona todos los plugins:

```php
// En MetricsCollector.php
protected function analyzeNodeIssues($node) {
  // Usa el plugin manager para ejecutar todos los checkers
  return $this->checkerManager->analyzeNode($node);
}
```

### 2. Ejecución Automática

El manager:
1. Descubre todos los plugins disponibles
2. Los ordena por peso (weight)
3. Ejecuta cada checker habilitado
4. Recopila todos los issues
5. Retorna el array completo

### 3. Checkers Individuales

Cada plugin implementa:

```php
/**
 * @EdaitorialChecker(
 *   id = "typos",
 *   label = @Translation("Typos Checker"),
 *   description = @Translation("Detects common typos"),
 *   category = "content",
 *   weight = 20
 * )
 */
class TyposChecker extends EdaitorialCheckerBase {
  
  public function analyze(NodeInterface $node) {
    $issues = [];
    
    // Lógica del checker
    
    return $issues;
  }
  
}
```

---

## 🎯 Resultado de Análisis

### Test Node 25: "Teh Complete Guide to SEO"

**Total Issues:** 5

#### SEO (1 issue):
- ✗ Content too short: 83 words [Medium]

#### Content (4 issues):
- ✗ Possible typos in title: Teh → the [Medium]
- ✗ 8 possible typos detected [Medium]
- ✗ Repeated words found: The The [Low]
- ✗ Suggestion: Consider using more active voice [Low]

**Checkers Ejecutados:**
1. ✅ SEO Checker → 1 issue
2. ✅ Broken Links Checker → 0 issues (no links broken)
3. ✅ Typos Checker → 3 issues (typos in title + body + repeated)
4. ✅ Suggestions Checker → 1 issue (active voice)

---

## 🔌 Crear un Nuevo Plugin

### Paso 1: Crear el archivo del plugin

```php
<?php

namespace Drupal\edaitorial\Plugin\EdaitorialChecker;

use Drupal\edaitorial\Plugin\EdaitorialCheckerBase;
use Drupal\node\NodeInterface;

/**
 * My custom checker.
 *
 * @EdaitorialChecker(
 *   id = "my_checker",
 *   label = @Translation("My Checker"),
 *   description = @Translation("Does something cool"),
 *   category = "content",
 *   weight = 40
 * )
 */
class MyChecker extends EdaitorialCheckerBase {

  /**
   * {@inheritdoc}
   */
  public function analyze(NodeInterface $node) {
    $issues = [];
    
    // Tu lógica aquí
    
    return $issues;
  }

}
```

### Paso 2: Limpiar caché

```bash
ddev drush cr
```

### Paso 3: ¡Listo!

El plugin será descubierto automáticamente y ejecutado.

---

## 🎛️ Habilitar/Deshabilitar Checkers

Los checkers se pueden habilitar/deshabilitar mediante configuración:

```php
$config = \Drupal::configFactory()->getEditable('edaitorial.settings');
$config->set('enabled_checkers', [
  'seo',
  'broken_links',
  'typos',
  // 'suggestions', // Deshabilitado
])->save();
```

Por defecto, **todos** los checkers están habilitados.

---

## 📊 Comparación: Antes vs Ahora

### ANTES: Código Monolítico
```php
// Un método gigante con 400+ líneas
protected function analyzeNodeIssues($node) {
  $issues = [];
  
  // SEO checks (100 lines)
  // ...
  
  // Accessibility checks (100 lines)
  // ...
  
  // Link checks (100 lines)
  // ...
  
  // Content checks (100 lines)
  // ...
  
  return $issues;
}
```

**Problemas:**
- ❌ Difícil de mantener
- ❌ No extensible
- ❌ No se pueden deshabilitar checks individuales
- ❌ Código mezclado
- ❌ Testing difícil

---

### AHORA: Sistema de Plugins
```php
// Método simple
protected function analyzeNodeIssues($node) {
  return $this->checkerManager->analyzeNode($node);
}

// Plugins separados
- SeoChecker.php (100 lines)
- BrokenLinksChecker.php (150 lines)
- TyposChecker.php (120 lines)
- SuggestionsChecker.php (130 lines)
```

**Beneficios:**
- ✅ Fácil de mantener
- ✅ Extensible (añadir nuevos plugins)
- ✅ Se pueden habilitar/deshabilitar individualmente
- ✅ Código organizado por responsabilidad
- ✅ Testing sencillo (test cada plugin)
- ✅ Otros módulos pueden añadir checkers

---

## 🎯 Categorías de Checkers

| Categoría | Checkers | Total Issues |
|-----------|----------|--------------|
| **SEO** | seo, broken_links | ~15 tipos |
| **Content** | typos, suggestions | ~12 tipos |
| **Accessibility** | (incluido en otros) | ~8 tipos |
| **Security** | (incluido en broken_links) | ~1 tipo |

---

## 🔍 Ejemplo: Detección de Typos

### Input:
```
Title: "Teh Complete Guide"
Body: "We recieve many questions. This is definately true."
```

### Output:
```
✗ Possible typos in title: Teh → the [Medium]
✗ 2 possible typos detected: recieve → receive, definately → definitely [Low]
```

### Diccionario de Typos:
El checker incluye 50+ typos comunes:
- teh → the
- recieve → receive
- definately → definitely
- goverment → government
- alot → a lot
- beleive → believe
- becuase → because
- ... (y más)

---

## 🚀 Ventajas del Sistema de Plugins

### 1. **Modularidad**
Cada checker es independiente y se puede desarrollar/mantener por separado.

### 2. **Extensibilidad**
Otros módulos pueden añadir sus propios checkers sin modificar edAItorial:

```php
// En otro módulo: my_module/src/Plugin/EdaitorialChecker/MyChecker.php
/**
 * @EdaitorialChecker(
 *   id = "my_custom_checker",
 *   ...
 * )
 */
class MyChecker extends EdaitorialCheckerBase { ... }
```

### 3. **Performance**
Se pueden deshabilitar checkers pesados si no se necesitan.

### 4. **Testing**
Cada plugin se puede testear independientemente:

```php
$checker = new TyposChecker(...);
$issues = $checker->analyze($node);
$this->assertCount(3, $issues);
```

### 5. **Reutilización**
La clase base `EdaitorialCheckerBase` proporciona helpers comunes:

```php
// Disponible en todos los plugins
$body = $this->getTextContent($node);
$enabled = $this->isEnabled();
```

---

## 📖 API del Plugin

### Métodos Principales:

```php
interface EdaitorialCheckerInterface {
  
  /**
   * Analiza un nodo.
   *
   * @return array
   *   Array of issues: [
   *     'description' => string,
   *     'type' => 'SEO'|'Content'|'Accessibility'|'Security',
   *     'severity' => 'Critical'|'High'|'Medium'|'Low',
   *     'impact' => 'High'|'Medium'|'Low',
   *   ]
   */
  public function analyze(NodeInterface $node);
  
  /**
   * @return string Category: 'seo'|'content'|'accessibility'|'security'
   */
  public function getCategory();
  
  /**
   * @return string Human-readable label
   */
  public function getLabel();
  
  /**
   * @return bool TRUE if enabled
   */
  public function isEnabled();
  
}
```

### Helpers en Base Class:

```php
abstract class EdaitorialCheckerBase {
  
  /**
   * Get text content from node (searches multiple field names).
   */
  protected function getTextContent($node);
  
}
```

---

## 🎉 Resumen

### Plugins Creados: 4

1. **SEO Checker** - Análisis SEO completo
2. **Broken Links Checker** - Links rotos en HTML y fields
3. **Typos Checker** ⭐ - Detección de errores ortográficos
4. **Suggestions Checker** ⭐ - Sugerencias de mejora

### Checks Totales: 30+

- SEO: ~15 checks
- Content: ~12 checks
- Accessibility: ~8 checks
- Security: ~1 check

### Características:

✅ Sistema de plugins modular  
✅ Fácilmente extensible  
✅ Checkers pueden habilitarse/deshabilitarse  
✅ Detección de typos implementada  
✅ Sugerencias de contenido implementadas  
✅ Código organizado y mantenible  
✅ Compatible con otros módulos  
✅ Testing simplificado  

---

**Fecha:** 2026-01-27  
**Versión:** 3.0 (Plugin System)  
**Plugins:** 4  
**Checks:** 30+  
**Estado:** ✅ Completamente funcional
