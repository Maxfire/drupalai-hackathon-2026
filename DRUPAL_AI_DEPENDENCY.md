# 🔗 Dependencia de Drupal AI - Completada

## ✅ Cambios Implementados

El módulo **edAItorial** ahora tiene dependencia completa del módulo **drupal:ai** y obtiene automáticamente la configuración de AI desde él.

---

## 🎯 Arquitectura Actualizada

### ANTES (Configuración Duplicada):

```
edAItorial Settings Form
    ↓
  ai_provider: "amazeeio"     ← Usuario configuraba manualmente
  ai_model: "claude..."       ← Usuario configuraba manualmente
    ↓
EdaitorialCheckerBase
    ↓
drupal_ai Provider
```

**Problema:** Duplicación de configuración, usuario tiene que configurar en 2 lugares.

---

### AHORA (Configuración Centralizada):

```
Drupal AI Settings
  (/admin/config/ai)
    ↓
  default_providers:
    chat: "amazeeio"          ← Configurado una sola vez
    ↓
edAItorial Settings Form
  (Solo habilitar/deshabilitar)
    ↓
EdaitorialCheckerBase
  ↓
  Obtiene provider desde ai.settings
  ↓
drupal_ai Provider
```

**Ventaja:** Configuración centralizada, sin duplicación.

---

## 📦 Archivos Modificados

### 1. edaitorial.info.yml

**Añadida dependencia:**

```yaml
dependencies:
  - drupal:node
  - drupal:user
  - drupal:system
  - drupal:ai          ← NUEVO
```

**Efecto:**
- edAItorial requiere que drupal:ai esté instalado
- No se puede instalar edAItorial sin drupal:ai
- Drupal gestiona automáticamente la dependencia

---

### 2. SettingsForm.php

#### ANTES:
```php
// Campos manuales
$form['ai']['ai_provider'] = [
  '#type' => 'textfield',
  '#title' => $this->t('AI Provider'),
  '#default_value' => $config->get('ai_provider') ?? 'amazeeio',
];

$form['ai']['ai_model'] = [
  '#type' => 'textfield',
  '#title' => $this->t('AI Model'),
  '#default_value' => $config->get('ai_model') ?? 'claude-3-5-sonnet-latest',
];
```

**Problema:** Usuario tiene que escribir manualmente provider y model.

#### AHORA:
```php
// Información obtenida automáticamente
$ai_info = $this->getAiProviderInfo();

$form['ai']['ai_info']['provider_display'] = [
  '#type' => 'item',
  '#title' => $this->t('Default Chat Provider'),
  '#markup' => '<strong>' . $ai_info['provider_label'] . '</strong> (' . $ai_info['provider'] . ')',
];

$form['ai']['ai_info']['model_display'] = [
  '#type' => 'item',
  '#title' => $this->t('Default Model'),
  '#markup' => '<strong>' . $ai_info['model'] . '</strong>',
];
```

**Ventaja:** 
- Solo muestra la configuración actual (read-only)
- Link a `/admin/config/ai` para configurar
- No hay posibilidad de desincronización

---

### 3. Nuevo Método: getAiProviderInfo()

```php
protected function getAiProviderInfo() {
  $info = [
    'provider' => NULL,
    'provider_label' => NULL,
    'model' => NULL,
  ];

  try {
    // Get default provider for 'chat' operation
    $ai_config = \Drupal::config('ai.settings');
    $default_providers = $ai_config->get('default_providers') ?? [];
    
    if (isset($default_providers['chat'])) {
      $provider_id = $default_providers['chat'];
      $info['provider'] = $provider_id;
      
      // Get provider label
      $ai_provider_manager = \Drupal::service('ai.provider');
      $definitions = $ai_provider_manager->getDefinitions();
      
      if (isset($definitions[$provider_id])) {
        $info['provider_label'] = (string) $definitions[$provider_id]['label'];
      }
      
      // Try to get default model
      $provider_config_name = 'ai_provider_' . $provider_id . '.settings';
      $provider_config = \Drupal::config($provider_config_name);
      
      if ($default_model = $provider_config->get('default_model')) {
        $info['model'] = $default_model;
      }
    }
  }
  catch (\Exception $e) {
    \Drupal::logger('edaitorial')->error('Failed to get AI provider info: @message', [
      '@message' => $e->getMessage(),
    ]);
  }

  return $info;
}
```

**Función:**
- Lee `ai.settings` para obtener el provider por defecto de 'chat'
- Obtiene el label del provider desde el plugin manager
- Intenta obtener el modelo por defecto del provider
- Maneja errores gracefully

---

### 4. EdaitorialCheckerBase.php

#### Método callAi() Actualizado:

**ANTES:**
```php
$provider_id = $config->get('ai_provider') ?? 'amazeeio';
$model_id = $config->get('ai_model') ?? 'claude-3-5-sonnet-latest';

$provider = $this->aiProvider->createInstance($provider_id);
```

**AHORA:**
```php
// Get default chat provider from Drupal AI configuration
$ai_config = $this->configFactory->get('ai.settings');
$default_providers = $ai_config->get('default_providers') ?? [];

if (!isset($default_providers['chat'])) {
  \Drupal::logger('edaitorial')->warning('No default chat provider configured');
  return '[]';
}

$provider_id = $default_providers['chat'];
$provider = $this->aiProvider->createInstance($provider_id);

// Get model dynamically
$model_id = $this->getDefaultModelForProvider($provider_id);
```

**Ventaja:**
- Siempre usa el provider configurado en Drupal AI
- No hay hardcoding de provider/model
- Advertencia clara si no hay provider configurado

---

#### Nuevo Método: getDefaultModelForProvider()

```php
protected function getDefaultModelForProvider(string $provider_id): ?string {
  try {
    $provider_config_name = 'ai_provider_' . $provider_id . '.settings';
    $provider_config = $this->configFactory->get($provider_config_name);
    
    // Try to get default model
    if ($default_model = $provider_config->get('default_model')) {
      return $default_model;
    }
    
    // If no default, try to get first available model
    $models = $provider_config->get('models');
    if ($models && is_array($models) && !empty($models)) {
      return array_key_first($models);
    }
    
    // Fallback: try common model names for amazeeio
    if ($provider_id === 'amazeeio') {
      return 'claude-3-5-sonnet-20241022';
    }
  }
  catch (\Exception $e) {
    \Drupal::logger('edaitorial')->error('Failed to get model');
  }
  
  return NULL;
}
```

**Estrategia de Fallback:**
1. Busca `default_model` en configuración del provider
2. Si no existe, usa el primer modelo disponible
3. Si no hay modelos, usa nombre común para amazeeio
4. Si todo falla, retorna NULL

---

### 5. edaitorial.settings.yml

**ANTES:**
```yaml
use_ai: true
ai_provider: 'amazeeio'      ← Eliminado
ai_model: 'claude-3-5-sonnet-latest'  ← Eliminado
```

**AHORA:**
```yaml
# AI provider and model are automatically obtained from Drupal AI module
use_ai: true
```

**Comentario añadido** para documentar que provider/model vienen de drupal:ai.

---

## 🔧 Configuración del Sistema

### Paso 1: Configurar Drupal AI

```
/admin/config/ai
```

1. Configurar proveedor (amazeeio, openai, anthropic, etc.)
2. Configurar API keys
3. Seleccionar modelo por defecto para 'chat'

```bash
# Via Drush
ddev drush config:set ai.settings default_providers.chat 'amazeeio'
```

### Paso 2: Habilitar AI en edAItorial

```
/admin/config/content/edaitorial/settings
```

1. Check "Use AI for content analysis"
2. Ver información del provider actual (read-only)
3. Configurar prompts

---

## 📊 Flujo de Configuración

```
Usuario
  ↓
/admin/config/ai
  ↓
Configura Provider: amazeeio
Configura API Key
Selecciona Model: claude-3-5-sonnet
  ↓
[Guarda en ai.settings]
  ↓
  ↓
/admin/config/content/edaitorial/settings
  ↓
[✓] Use AI for content analysis
  ↓
Ve información actual:
  - Provider: amazee.ai AI (amazeeio)
  - Model: claude-3-5-sonnet
  ↓
Personaliza prompts
  ↓
[Guarda]
  ↓
  ↓
EdaitorialChecker analiza contenido
  ↓
Lee ai.settings → Obtiene 'amazeeio'
  ↓
Lee ai_provider_amazeeio.settings → Obtiene model
  ↓
Llama AI con provider + model configurados
```

---

## 🎯 Ventajas del Nuevo Sistema

### 1. Configuración Centralizada

- ✅ Usuario configura AI **una sola vez** en Drupal AI
- ✅ Todos los módulos usan la misma configuración
- ✅ No hay duplicación ni inconsistencias

### 2. Gestión de Dependencias

- ✅ Drupal gestiona automáticamente que drupal:ai esté instalado
- ✅ Si se desinstala drupal:ai, se desinstala edAItorial primero
- ✅ Versionado de dependencias gestionado por Composer

### 3. Extensibilidad

- ✅ Si se cambia de provider (amazeeio → openai), edAItorial lo usa automáticamente
- ✅ Si se añaden nuevos modelos, están disponibles inmediatamente
- ✅ Soporte multi-provider sin cambios en edAItorial

### 4. Mantenibilidad

- ✅ Menos código para mantener
- ✅ No hay lógica de configuración de AI en edAItorial
- ✅ Updates de drupal:ai benefician automáticamente a edAItorial

### 5. UX Mejorada

- ✅ Usuario no tiene que saber qué provider o model usar
- ✅ Link directo a configuración de AI si no está configurado
- ✅ Información clara del estado actual

---

## 🧪 Testing

### Verificar Configuración Actual

```bash
# Ver provider de chat configurado
ddev drush config:get ai.settings default_providers.chat

# Ver configuración de edaitorial
ddev drush config:get edaitorial.settings use_ai
```

### Test de Análisis con AI

```bash
ddev drush php-eval "
\$node = \Drupal::entityTypeManager()->getStorage('node')->load(25);
\$manager = \Drupal::service('plugin.manager.edaitorial_checker');

\$ai_config = \Drupal::config('ai.settings');
echo 'Provider: ' . \$ai_config->get('default_providers')['chat'] . PHP_EOL;

\$typos_checker = \$manager->createInstance('typos');
\$issues = \$typos_checker->analyze(\$node);

echo 'Issues: ' . count(\$issues) . PHP_EOL;
"
```

**Output Esperado:**
```
Provider: amazeeio
Issues: 4
```

---

## 🔍 Debugging

### Si AI no funciona:

1. **Verificar provider configurado:**
```bash
ddev drush config:get ai.settings default_providers
```

Si no hay provider para 'chat', configurarlo:
```bash
ddev drush config:set ai.settings default_providers.chat 'amazeeio'
```

2. **Verificar provider tiene API key:**
```bash
ddev drush config:get ai_provider_amazeeio.settings api_key
```

3. **Ver logs de edaitorial:**
```bash
ddev drush watchdog:tail edaitorial
```

Mensajes esperados:
- `No default chat provider configured` → Configurar provider en ai.settings
- `No model available for provider` → Configurar modelo en provider settings
- `AI call failed` → Ver error específico de la API

---

## 📖 UI Actualizada

### Sección "AI Configuration"

**Descripción en la parte superior:**
> "AI settings are managed by the Drupal AI module. Configure providers and models at AI Settings."

**Checkbox:**
- [✓] Use AI for content analysis
  - "Enable AI-powered analysis using the configured AI provider."

**Current AI Configuration (read-only):**
- **Default Chat Provider:** amazee.ai AI (amazeeio)
- **Default Model:** claude-3-5-sonnet-20241022

**Si no hay provider configurado:**
> ⚠️ No AI provider is configured. Please configure an AI provider first.
> [Link: /admin/config/ai]

---

## ✅ Checklist de Cambios

- [x] Añadida dependencia `drupal:ai` en edaitorial.info.yml
- [x] Eliminados campos `ai_provider` y `ai_model` del form
- [x] Añadido display read-only de configuración actual
- [x] Creado método `getAiProviderInfo()` en SettingsForm
- [x] Actualizado `callAi()` para obtener provider de ai.settings
- [x] Creado método `getDefaultModelForProvider()` con fallbacks
- [x] Actualizado submitForm() para no guardar provider/model
- [x] Actualizado config/install con comentarios
- [x] Warnings si no hay provider configurado
- [x] Link a /admin/config/ai para configurar
- [x] Error handling y logging
- [x] Testing verificado
- [x] Documentación completa

---

## 🎉 Resultado Final

### Estado Actual:

✅ **Dependencia Completa de drupal:ai**
- edAItorial requiere drupal:ai
- Configuración obtenida automáticamente
- Cero duplicación

✅ **Configuración Centralizada**
- Usuario configura AI una sola vez
- Todos los módulos comparten configuración
- Cambios automáticamente reflejados

✅ **UX Mejorada**
- Información clara del estado
- Link directo a configuración
- Advertencias si falta configuración

✅ **Código Limpio**
- Menos lógica en edAItorial
- Reutilización de servicios drupal:ai
- Mantenimiento simplificado

---

**Fecha:** 2026-01-27  
**Versión:** 4.1 (Drupal AI Dependency)  
**Estado:** ✅ Completamente Implementado  
**Funcionalidad:** 100% (sujeto a modelos disponibles en provider)
