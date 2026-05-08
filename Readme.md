# Construct Addon SDK v2 — Полный справочник для ИИ-агентов

> **ВАЖНО:** Этот документ содержит только методы и поля, реально существующие в SDK v2.  
> SDK v1 не поддерживается начиная с Construct 3 r450+.  
> Официальная документация: https://www.construct.net/make-games/manuals/addon-sdk

---

## Содержание

1. [Типы аддонов](#1-типы-аддонов)
2. [Структура файлов](#2-структура-файлов)
3. [addon.json — поля и обязательные значения](#3-addonjson)
4. [aces.json — Actions / Conditions / Expressions](#4-acesjson)
5. [lang/en-US.json](#5-angen-usjson)
6. [EDITOR SIDE — классы редактора (plugin.js / behavior.js)](#6-editor-side)
7. [RUNTIME SIDE — классы рантайма (c3runtime/)](#7-runtime-side)
8. [DOM-side scripting (domSide.js)](#8-dom-side-scripting)
9. [Wrapper Extensions (C++)](#9-wrapper-extensions)
10. [Effect SDK](#10-effect-sdk)
11. [Theme SDK](#11-theme-sdk)
12. [Частые ошибки ИИ-агентов](#12-частые-ошибки-ии-агентов)

---

## 1. Типы аддонов

| Тип | Назначение | Ключевые файлы |
|---|---|---|
| `plugin` | Новый объект (мировой или глобальный) | `plugin.js`, `type.js`, `instance.js`, `c3runtime/` |
| `behavior` | Поведение для существующих объектов | `behavior.js`, `type.js`, `instance.js`, `c3runtime/` |
| `effect` | WebGL/WebGPU шейдер | `effect.fx` (GLSL), `effect.wgsl` (WGSL) |
| `theme` | CSS-тема редактора | `theme.css` |

---

## 2. Структура файлов

### Plugin

```
my-plugin/
├── addon.json            # метаданные аддона
├── aces.json             # определения Actions/Conditions/Expressions
├── icon.svg              # иконка
├── plugin.js             # Editor side: класс плагина
├── type.js               # Editor side: класс типа объекта
├── instance.js           # Editor side: класс экземпляра
├── lang/
│   └── en-US.json        # строки локализации
└── c3runtime/
    ├── main.js           # импортирует остальные runtime-файлы
    ├── plugin.js         # Runtime: класс плагина
    ├── type.js           # Runtime: класс типа
    ├── instance.js       # Runtime: класс экземпляра
    ├── actions.js        # Runtime: реализации действий
    ├── conditions.js     # Runtime: реализации условий
    ├── expressions.js    # Runtime: реализации выражений
    └── domSide.js        # (опц.) DOM-side скрипт для DOM/Worker-режима
```

### Behavior

```
my-behavior/
├── addon.json
├── aces.json
├── icon.svg
├── behavior.js           # Editor side: класс поведения
├── type.js               # Editor side: класс типа поведения
├── instance.js           # Editor side: класс экземпляра поведения
├── lang/en-US.json
└── c3runtime/
    ├── main.js
    ├── behavior.js
    ├── type.js
    ├── instance.js
    ├── actions.js
    ├── conditions.js
    └── expressions.js
```

### c3runtime/main.js (обязательное содержимое)

```js
import "./plugin.js";
import "./type.js";
import "./instance.js";
import "./conditions.js";
import "./actions.js";
import "./expressions.js";
```

---

## 3. addon.json

### Обязательные поля для plugin/behavior

```json
{
    "$schema": "../plugin.addon.schema.json",
    "is-c3-addon": true,
    "sdk-version": 2,
    "type": "plugin",
    "name": "My Plugin",
    "id": "MyCompany_MyPlugin",
    "version": "1.0.0.0",
    "author": "Name",
    "website": "https://example.com",
    "documentation": "https://example.com",
    "description": "Short description.",
    "editor-scripts": [
        "plugin.js",
        "type.js",
        "instance.js"
    ],
    "file-list": [
        "c3runtime/main.js",
        "c3runtime/plugin.js",
        "c3runtime/type.js",
        "c3runtime/instance.js",
        "c3runtime/conditions.js",
        "c3runtime/actions.js",
        "c3runtime/expressions.js",
        "lang/en-US.json",
        "aces.json",
        "addon.json",
        "icon.svg",
        "instance.js",
        "plugin.js",
        "type.js"
    ]
}
```

**Для behavior** — `editor-scripts` содержит `["behavior.js", "type.js", "instance.js"]`, а `type` = `"behavior"`.

**Опциональные поля addon.json:**

| Поле | Тип | Описание |
|---|---|---|
| `min-construct-version` | `string` | Минимальная версия, например `"r401"` |
| `supports-worker-mode` | `boolean` | Поддержка Web Worker режима |

---

## 4. aces.json

Структура aces.json — объект, где **ключи** — это id категорий АСЕ (строки, используемые в lang как `aceCategories`). Пустая строка `""` означает категорию по умолчанию.

```json
{
    "$schema": "../aces.schema.json",

    "my-category": {
        "conditions": [ /* ... */ ],
        "actions": [ /* ... */ ],
        "expressions": [ /* ... */ ]
    }
}
```

### Condition (условие)

```json
{
    "id": "compare-speed",
    "scriptName": "CompareSpeed",
    "isTrigger": false,
    "isFakeTrigger": false,
    "isStatic": false,
    "isLooping": false,
    "isInvertible": true,
    "isCompatibleWithTriggers": false,
    "highlight": false,
    "isDeprecated": false,
    "params": [
        { "id": "comparison", "type": "cmp" },
        { "id": "speed", "type": "number" }
    ]
}
```

**Триггер:**
```json
{
    "id": "on-hit-solid",
    "scriptName": "OnHitSolid",
    "isTrigger": true
}
```

### Action (действие)

```json
{
    "id": "set-speed",
    "scriptName": "SetSpeed",
    "isAsync": false,
    "highlight": false,
    "isDeprecated": false,
    "params": [
        { "id": "speed", "type": "number" }
    ]
}
```

### Expression (выражение)

```json
{
    "id": "speed",
    "expressionName": "Speed",
    "returnType": "number",
    "highlight": false,
    "isDeprecated": false,
    "isVariadicParameters": false,
    "params": [
        { "id": "value", "type": "number" }
    ]
}
```

**`returnType`** может быть только: `"number"`, `"string"`, `"any"`.

### Типы параметров (для условий и действий)

| Тип | Описание |
|---|---|
| `"number"` | Числовое выражение |
| `"string"` | Строковое выражение |
| `"any"` | Любое выражение |
| `"boolean"` | Булево (true/false) |
| `"cmp"` | Оператор сравнения (=, ≠, <, >, и т.д.) |
| `"combo"` | Выпадающий список; требует `"items": ["item1", "item2"]` |
| `"combo-grouped"` | Группированный список; требует `"itemGroups"` |
| `"object"` | Ссылка на объект |
| `"objectname"` | Имя объекта (строка) |
| `"layer"` | Слой |
| `"layout"` | Компоновка |
| `"keyb"` | Клавиша клавиатуры |
| `"instancevar"` | Переменная экземпляра |
| `"instancevarbool"` | Булева переменная экземпляра |
| `"eventvar"` | Переменная события |
| `"eventvarbool"` | Булева переменная события |
| `"animation"` | Анимация |
| `"objinstancevar"` | Переменная экземпляра объекта |
| `"projectfile"` | Файл проекта |

**Типы параметров выражений:** только `"number"`, `"string"`, `"any"`.

---

## 5. lang/en-US.json

```json
{
    "$schema": "../../plugin.lang.schema.json",
    "languageTag": "en-US",
    "fileDescription": "Strings for MyPlugin.",
    "text": {
        "plugins": {
            "mycompany_myplugin": {
                "name": "My Plugin",
                "description": "An example plugin.",
                "help-url": "https://example.com",
                "properties": {
                    "my-property": {
                        "name": "My Property",
                        "desc": "Description of the property."
                    }
                },
                "aceCategories": {
                    "my-category": "My Category"
                },
                "conditions": {
                    "compare-speed": {
                        "list-name": "Compare speed",
                        "display-text": "Speed {0} {1}",
                        "description": "Compare the speed value.",
                        "params": {
                            "comparison": { "name": "Comparison", "desc": "Comparison operator." },
                            "speed": { "name": "Speed", "desc": "Speed to compare to." }
                        }
                    }
                },
                "actions": {
                    "set-speed": {
                        "list-name": "Set speed",
                        "display-text": "Set speed to {0}",
                        "description": "Set the speed value.",
                        "params": {
                            "speed": { "name": "Speed", "desc": "New speed value." }
                        }
                    }
                },
                "expressions": {
                    "speed": {
                        "description": "Get current speed.",
                        "translated-name": "Speed",
                        "params": {}
                    }
                }
            }
        }
    }
}
```

**Для behavior** — вместо `"plugins"` используется `"behaviors"`.  
Ключ — это **ID в нижнем регистре** (`PLUGIN_ID.toLowerCase()`).

---

## 6. Editor Side

Глобальная переменная: `const SDK = globalThis.SDK;`

### 6.1 SDK.IPluginBase (plugin.js)

Базовый класс для редакторной части плагина.

```js
const PLUGIN_CLASS = SDK.Plugins.MyCompany_MyPlugin = class MyPlugin extends SDK.IPluginBase
{
    constructor()
    {
        super(PLUGIN_ID);   // PLUGIN_ID — строка, уникальный идентификатор

        SDK.Lang.PushContext("plugins." + PLUGIN_ID.toLowerCase());

        // Обязательные вызовы на this._info:
        this._info.SetName(globalThis.lang(".name"));
        this._info.SetDescription(globalThis.lang(".description"));
        this._info.SetCategory(PLUGIN_CATEGORY);
        this._info.SetAuthor("Author Name");
        this._info.SetHelpUrl(globalThis.lang(".help-url"));
        this._info.SetRuntimeModuleMainScript("c3runtime/main.js");

        // Опциональные вызовы на this._info:
        this._info.SetIsSingleGlobal(true);        // плагин — одиночный глобальный
        this._info.SetPluginType("world");          // "world" для объектов в макете
        this._info.SetIsResizable(true);
        this._info.SetIsRotatable(true);
        this._info.SetHasImage(true);               // плагин имеет редактируемое изображение
        this._info.SetSupportsEffects(true);
        this._info.SetSupportsChangingSampling(true);
        this._info.SetMustPreDraw(true);
        this._info.SetDOMSideScripts(["c3runtime/domSide.js"]);  // для DOM-плагинов

        // Добавление зависимостей (wrapper extension DLL)
        this._info.AddFileDependency({
            filename: "MyExtension_x64.ext.dll",
            type: "wrapper-extension",
            platform: "windows-x64"   // "windows-x86", "windows-arm64", "linux-x64"
        });

        SDK.Lang.PushContext(".properties");

        this._info.SetProperties([
            new SDK.PluginProperty("integer", "my-prop", 0),
            new SDK.PluginProperty("float",   "speed",   100.0),
            new SDK.PluginProperty("text",    "label",   "Hello"),
            new SDK.PluginProperty("longtext","body",    ""),
            new SDK.PluginProperty("check",   "enabled", true),
            new SDK.PluginProperty("combo",   "mode", { items: ["option-a", "option-b"] }),
            new SDK.PluginProperty("color",   "tint",  [1, 0, 0]),
            new SDK.PluginProperty("link",    "edit-image", {
                linkCallback: function(param) { /* param — SDK.ITypeBase или SDKEditorInstance */ },
                callbackType: "once-for-type"      // или "for-each-instance"
            }),
        ]);

        SDK.Lang.PopContext();  // .properties
        SDK.Lang.PopContext();
    }
};

PLUGIN_CLASS.Register(PLUGIN_ID, PLUGIN_CLASS);
```

**Типы свойств (`SDK.PluginProperty`):**

| Тип | Описание |
|---|---|
| `"integer"` | Целое число |
| `"float"` | Вещественное число |
| `"percent"` | Вещественное 0–1, отображается как % |
| `"text"` | Короткая строка |
| `"longtext"` | Длинная строка с кнопкой |
| `"check"` | Булево (флажок) |
| `"combo"` | Выпадающий список, 3й аргумент: `{ items: ["a","b"] }` |
| `"color"` | Цвет RGB, 3й аргумент: `[r, g, b]` (0–1) |
| `"link"` | Ссылка-кнопка, 3й аргумент: `{ linkCallback, callbackType }` |
| `"object"` | Ссылка на объект |

### 6.2 SDK.IBehaviorBase (behavior.js)

```js
const BEHAVIOR_CLASS = SDK.Behaviors.MyBehavior = class MyBehavior extends SDK.IBehaviorBase
{
    constructor()
    {
        super(BEHAVIOR_ID);

        SDK.Lang.PushContext("behaviors." + BEHAVIOR_ID.toLowerCase());

        this._info.SetName(globalThis.lang(".name"));
        this._info.SetDescription(globalThis.lang(".description"));
        this._info.SetCategory(BEHAVIOR_CATEGORY);
        this._info.SetAuthor("Author");
        this._info.SetHelpUrl(globalThis.lang(".help-url"));
        this._info.SetIsOnlyOneAllowed(true);    // только один экземпляр на объект
        this._info.SetRuntimeModuleMainScript("c3runtime/main.js");

        SDK.Lang.PushContext(".properties");
        this._info.SetProperties([ /* SDK.PluginProperty... */ ]);
        SDK.Lang.PopContext();
        SDK.Lang.PopContext();
    }
};

BEHAVIOR_CLASS.Register(BEHAVIOR_ID, BEHAVIOR_CLASS);
```

### 6.3 SDK.IWorldInstanceBase (editor instance.js для world-плагинов)

```js
PLUGIN_CLASS.Instance = class MyInstance extends SDK.IWorldInstanceBase
{
    constructor(sdkType, inst)  // inst: SDK.IWorldInstance
    {
        super(sdkType, inst);
        // this._inst — SDK.IWorldInstance
    }

    Release() { }
    OnCreate() { }
    OnPlacedInLayout() { }    // при размещении в редакторе
    OnPropertyChanged(id, value) { }
    LoadC2Property(name, valueString) { return false; }

    // Для world-плагинов с изображением:
    IsOriginalSizeKnown()  { return true; }
    GetOriginalWidth()     { return this.GetObjectType().GetImage().GetWidth(); }
    GetOriginalHeight()    { return this.GetObjectType().GetImage().GetHeight(); }
    OnMakeOriginalSize()   { this._inst.SetSize(w, h); }

    HasDoubleTapHandler()  { return true; }
    OnDoubleTap()          { /* open editor */ }

    Draw(iRenderer, iDrawParams)
    {
        // iRenderer: SDK.Gfx.IWebGLRenderer
        // iDrawParams: SDK.Gfx.IDrawParams
        const iLayoutView = iDrawParams.GetLayoutView();
        const quad = this._inst.GetQuad();          // SDK.Quad
        const texture = this.GetTexture(image);     // WebGLTexture | null

        this._inst.ApplyBlendMode(iRenderer);
        iRenderer.SetTextureFillMode();
        iRenderer.SetColorFillMode();
        iRenderer.SetTexture(texture, this._inst.GetActiveSampling());
        iRenderer.SetColor(this._inst.GetColor());
        iRenderer.SetColorRgba(r, g, b, a);
        iRenderer.Quad(quad);
        iRenderer.Quad3(quad, texRect);
        iRenderer.LineQuad(quad);
        iRenderer.SetAlphaBlend();
        iRenderer.ResetColor();
    }
}
```

**Методы `this._inst` (SDK.IWorldInstance / SDK.IObjectInstance):**

| Метод | Описание |
|---|---|
| `this._inst.SetSize(w, h)` | Установить размер |
| `this._inst.GetWidth()` | Ширина |
| `this._inst.GetHeight()` | Высота |
| `this._inst.SetOrigin(x, y)` | Установить точку привязки (0–1) |
| `this._inst.GetQuad()` | Получить SDK.Quad |
| `this._inst.GetAngle()` | Угол в радианах |
| `this._inst.GetColor()` | Цвет SDK.Color |
| `this._inst.GetActiveSampling()` | Режим сэмплирования |
| `this._inst.ApplyBlendMode(iRenderer)` | Применить режим смешивания |
| `this._inst.GetPropertyValue("prop-id")` | Получить значение свойства |
| `this._inst.SetPropertyValue("prop-id", val)` | Установить значение свойства |

**Методы `iLayoutView` (SDK.UI.ILayoutView):**

| Метод | Описание |
|---|---|
| `iLayoutView.GetZoomFactor()` | Коэффициент масштабирования |
| `iLayoutView.Refresh()` | Перерисовать вид |
| `iLayoutView.SetDeviceTransform(iRenderer)` | Переключить на device-пространство |
| `iLayoutView.SetDefaultTransform(iRenderer)` | Восстановить стандартный трансформ |
| `iLayoutView.LayoutToClientDeviceX(x)` | Координата X макета → device px |
| `iLayoutView.LayoutToClientDeviceY(y)` | Координата Y макета → device px |
| `iLayoutView.GetProject()` | SDK.IProject |
| `iLayoutView.GetActiveLayer()` | SDK.ILayer |

**Методы `SDK.Gfx.IWebGLText` (создаётся через `iRenderer.CreateRendererText()`):**

| Метод | Описание |
|---|---|
| `webglText.SetFontSize(pt)` | Размер шрифта в pt |
| `webglText.SetFontName(name)` | Имя шрифта |
| `webglText.SetText(str)` | Текст |
| `webglText.SetSize(w, h, zoom)` | Размер области + масштаб |
| `webglText.SetHorizontalAlignment("center")` | `"left"`, `"center"`, `"right"` |
| `webglText.SetVerticalAlignment("center")` | `"top"`, `"center"`, `"bottom"` |
| `webglText.SetTextureUpdateCallback(fn)` | Callback при обновлении текстуры |
| `webglText.GetTexture()` | Текстура (null если не готова) |
| `webglText.GetTexRect()` | Rect координат текстуры |
| `webglText.Release()` | Освободить ресурс |

### 6.4 SDK.IInstanceBase (editor instance.js для не-world плагинов)

```js
PLUGIN_CLASS.Instance = class MyInstance extends SDK.IInstanceBase
{
    constructor(sdkType, inst) { super(sdkType, inst); }
    Release() { }
    OnCreate() { }
    OnPropertyChanged(id, value) { }
    LoadC2Property(name, valueString) { return false; }
};
```

### 6.5 SDK.IBehaviorInstanceBase (editor instance.js для behavior)

```js
BEHAVIOR_CLASS.Instance = class MyBehaviorInstance extends SDK.IBehaviorInstanceBase
{
    constructor(sdkBehType, behInst) { super(sdkBehType, behInst); }
    Release() { }
    OnCreate() { }
    OnPropertyChanged(id, value) { }
    LoadC2Property(name, valueString) { return false; }
};
```

### 6.6 SDK.ITypeBase (type.js)

```js
PLUGIN_CLASS.Type = class MyType extends SDK.ITypeBase
{
    constructor(sdkPlugin, iObjectType) { super(sdkPlugin, iObjectType); }
    // Методы для плагинов с изображением:
    GetImage() { return this._iObjectType.GetDefaultAnimation().GetDefaultFrame().GetImage(); }
    GetObjectType() { return this._iObjectType; }
    EditImage() { this._iObjectType.GetDefaultAnimation().GetDefaultFrame().EditImage(); }
};
```

### 6.7 SDK.Lang

| Метод | Описание |
|---|---|
| `SDK.Lang.PushContext("plugins.myplugin")` | Установить контекст локализации |
| `SDK.Lang.PopContext()` | Снять контекст |
| `SDK.Lang.Get("full.key.path")` | Получить строку по полному пути |
| `globalThis.lang(".name")` | Получить строку относительно текущего контекста |

### 6.8 SDK.UI.Util

| Метод | Описание |
|---|---|
| `SDK.UI.Util.ShowLongTextPropertyDialog(text, title)` | Диалог редактирования длинного текста, возвращает `Promise<string\|null>` |
| `SDK.UI.Util.AddDragDropFileImportHandler(callback, opts)` | Регистрация обработчика drag-and-drop импорта |

**Опции `AddDragDropFileImportHandler`:**
```js
{
    isZipFormat: true,      // второй аргумент callback — SDK.IZipFile
    toLayoutView: true      // третий аргумент callback — содержит layoutView, layoutX, layoutY
}
```

**Сигнатура callback:**
```js
async function HandleMyFormat(droppedFileName, file, opts) {
    // file: SDK.IZipFile | Blob
    // opts.layoutView, opts.layoutX, opts.layoutY
    const zipFile = file; // SDK.IZipFile
    const entry = zipFile.GetEntry("data.json");       // ZipEntry | null
    const json  = await zipFile.ReadJson(entry);       // объект JSON
    const blob  = await zipFile.ReadBlob(entry);       // Blob
    return true; // или false, если формат не распознан
}
```

---

## 7. Runtime Side

Глобальная переменная: `const C3 = globalThis.C3;`

### 7.1 ISDKInstanceBase (c3runtime/instance.js — не-world плагин)

```js
class MyInstance extends globalThis.ISDKInstanceBase
{
    constructor()
    {
        super();
        // Для DOM-messaging (опц.): super({ domComponentId: "unique-id" })
        // Для wrapper-extension (опц.): super({ wrapperComponentId: "my-extension" })

        const properties = this._getInitProperties();
        // properties — массив или null
        // properties[0], properties[1], ... — значения свойств по порядку из addon.json
    }

    _release()
    {
        super._release();
    }

    _saveToJson()   { return { /* данные для сохранения */ }; }
    _loadFromJson(o) { /* восстановление из o */ }
}

C3.Plugins.MyPlugin.Instance = MyInstance;
```

**Поля и методы `ISDKInstanceBase` (доступны ч
