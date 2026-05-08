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
12. [Looping Conditions (циклы)](#12-looping-conditions-циклы-в-условиях)
13. [Утилиты C3](#13-утилиты-c3)
14. [Частые ошибки ИИ-агентов](#14-частые-ошибки-ии-агентов)

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

**Поля и методы `ISDKInstanceBase` (доступны через `this`):**

| Поле/Метод | Тип | Описание |
|---|---|---|
| `this.runtime` | IRuntime | Объект рантайма |
| `this.runtime.sdk` | ISDK | SDK-хелпер рантайма |
| `this.runtime.sdk.addLoadPromise(p)` | — | Задержать старт до разрешения Promise |
| `this.runtime.collisions` | ICollisions | Работа со столкновениями |
| `this.runtime.collisions.testOverlapSolid(inst)` | boolean | Проверить пересечение с Solid |
| `this.runtime.isPixelRoundingEnabled` | boolean | Pixel rounding включён |
| `this.runtime.assets` | IAssets | Загрузка ресурсов |
| `this.runtime.assets.loadImageAsset(imageInfo)` | — | Загрузить изображение |
| `this._getInitProperties()` | array\|null | Начальные значения свойств |
| `this._trigger(Cnds.MyTrigger)` | — | Сработать триггер |
| `this._postToDOM(msgId, data)` | — | Отправить сообщение на DOM-side (не ждёт ответа) |
| `this._postToDOMAsync(msgId, data)` | Promise | Отправить сообщение, ждёт ответа |
| `this._isWrapperExtensionAvailable()` | boolean | Доступен ли wrapper extension DLL |
| `this._sendWrapperExtensionMessageAsync(msgId, data)` | Promise | Отправить сообщение в C++ расширение |
| `this._addWrapperExtensionMessageHandler(msgId, fn)` | — | Получать сообщения от C++ расширения |
| `this.dispatchEvent(event)` | — | Отправить JS-событие (для addEventListener) |

### 7.2 ISDKWorldInstanceBase (c3runtime/instance.js — world-плагин)

Наследует все поля `ISDKInstanceBase`, плюс:

```js
class MyWorldInstance extends globalThis.ISDKWorldInstanceBase
{
    constructor() { super(); }

    _draw(renderer) {
        // renderer: IRenderer
        const imageInfo = this.objectType.getImageInfo();
        const texture   = imageInfo.getTexture(renderer);   // WebGLTexture | null
        const rcTex     = imageInfo.getTexRect();

        let quad = this.getBoundingQuad();                   // DOMQuad

        renderer.setTexture(texture, this.activeSampling);
        renderer.quad3(quad, rcTex);
        renderer.setDeviceTransform();
        renderer.setLayerTransform(layer);
    }
}
```

**Поля world-instance (через `this`):**

| Поле/Метод | Тип | Описание |
|---|---|---|
| `this.x` | number | Координата X |
| `this.y` | number | Координата Y |
| `this.width` | number | Ширина |
| `this.height` | number | Высота |
| `this.angle` | number | Угол (радианы) |
| `this.objectType` | IObjectType | Тип объекта |
| `this.layer` | ILayer | Слой |
| `this.layer.renderScale` | number | Масштаб рендера слоя |
| `this.layer.angle` | number | Угол слоя |
| `this.layer.layerToDrawSurface(x, y)` | [x, y] | Перевод координат |
| `this.activeSampling` | string | Режим сэмплирования |
| `this.runtime.isPixelRoundingEnabled` | boolean | Pixel rounding |
| `this.getBoundingQuad()` | DOMQuad | Ограничивающий квад |
| `this.getPosition()` | [x, y] | Позиция |
| `this.setPosition(x, y)` | — | Установить позицию |
| `this.offsetPosition(dx, dy)` | — | Сдвинуть позицию |
| `this.dt` | number | Delta-time |

**Методы `renderer` (IRenderer, runtime):**

| Метод | Описание |
|---|---|
| `renderer.setTexture(texture, sampling?)` | Установить текстуру |
| `renderer.quad3(quad, texRect)` | Нарисовать квад с текстурными координатами |
| `renderer.setDeviceTransform()` | Переключиться на device-координаты |
| `renderer.setLayerTransform(layer)` | Восстановить трансформ слоя |
| `renderer.createRendererText()` | Создать IRendererText |
| `renderer.loadTextureForImageInfo(imageInfo)` | Promise: загрузить текстуру |
| `renderer.releaseTextureForImageInfo(imageInfo)` | Освободить текстуру |

**Методы `IRendererText` (runtime):**

| Метод | Описание |
|---|---|
| `rt.sizePt` | Размер шрифта (свойство) |
| `rt.fontFace` | Имя шрифта (свойство) |
| `rt.text` | Текст (свойство) |
| `rt.setSize(w, h, zoom)` | Установить размер |
| `rt.getTexture()` | Текстура (null если не готова) |
| `rt.getTexRect()` | Rect текстурных координат |

### 7.3 ISDKDOMInstanceBase (c3runtime/instance.js — DOM-плагин)

```js
class MyDOMInstance extends globalThis.ISDKDOMInstanceBase
{
    constructor()
    {
        super({ domComponentId: DOM_COMPONENT_ID }); // DOM_COMPONENT_ID — уникальная строка

        this._createElement();  // создаёт DOM-элемент через domSide.js
    }

    _getElementState()
    {
        // Возвращает JSON-объект со всем состоянием элемента.
        // Вызывается автоматически при создании и при _updateElementState().
        return { "text": this._text };
    }

    _draw(renderer) { /* не используется для DOM-элементов */ }

    _saveToJson() { return { "text": this._text }; }
    _loadFromJson(o) {
        this._text = o["text"];
        this._updateElementState(); // обновить DOM после загрузки
    }
}
```

**Дополнительные методы `ISDKDOMInstanceBase`:**

| Метод | Описание |
|---|---|
| `this._createElement()` | Создать DOM-элемент (вызвать в конструкторе) |
| `this._updateElementState()` | Обновить состояние DOM (вызывает `_getElementState()`) |

### 7.4 ISDKBehaviorInstanceBase (c3runtime/instance.js поведения)

```js
class MyBehaviorInstance extends globalThis.ISDKBehaviorInstanceBase
{
    constructor()
    {
        super();

        const properties = this._getInitProperties();
        // properties[0], [1], ...

        this._setTicking(true);   // запустить _tick() каждый кадр
    }

    _tick()
    {
        const dt = this.instance.dt;           // delta-time
        const [x, y] = this.instance.getPosition();
        this.instance.offsetPosition(dx, dy);
        this.instance.setPosition(x, y);
        this.instance.angle;                   // угол (rad)
        this.runtime.collisions.testOverlapSolid(this.instance);
        this.dispatchEvent(new C3.Event("hit-solid"));
        this._trigger(C3.Behaviors.MyBehavior.Cnds.OnHitSolid);
    }

    _setTicking(enabled) { /* включить/выключить _tick() */ }

    _getDebuggerProperties()
    {
        return [{
            title: "$" + this.behaviorType.name,
            properties: [
                { name: "lang.key", value: this.speed, onedit: (v) => this.speed = v }
            ]
        }];
    }

    _saveToJson() { return { "s": this._speed }; }
    _loadFromJson(o) { this._speed = o["s"]; }
}
```

**Поля `ISDKBehaviorInstanceBase`:**

| Поле | Описание |
|---|---|
| `this.instance` | Связанный world-instance (IWorldInstance) |
| `this.runtime` | Рантайм |
| `this.behaviorType` | Тип поведения |

### 7.5 ISDKPluginBase / ISDKDOMPluginBase (c3runtime/plugin.js)

```js
// Обычный плагин:
C3.Plugins.MyPlugin = class MyPlugin extends globalThis.ISDKPluginBase
{
    constructor() { super(); }
};

// DOM-плагин (с DOM-элементами):
C3.Plugins.MyDOMPlugin = class MyDOMPlugin extends globalThis.ISDKDOMPluginBase
{
    constructor()
    {
        super({ domComponentId: DOM_COMPONENT_ID });

        // Форвардинг сообщений от DOM-side к экземплярам:
        this._addElementMessageHandler("click", (inst, e) => inst._onClick(e));
    }
};
```

**Метод `_addElementMessageHandler(msgId, fn)`** — принимает сообщения от `domSide.js → PostToRuntimeElement()` и направляет их в нужный экземпляр.

### 7.6 ISDKBehaviorBase (c3runtime/behavior.js)

```js
C3.Behaviors.MyBehavior = class MyBehavior extends globalThis.ISDKBehaviorBase
{
    constructor() { super(); }
};
```

### 7.7 ISDKObjectTypeBase (c3runtime/type.js)

```js
C3.Plugins.MyPlugin.Type = class MyType extends globalThis.ISDKObjectTypeBase
{
    constructor() { super(); }

    _onCreate()
    {
        // Для плагинов с изображением — загрузить ресурс:
        this.runtime.assets.loadImageAsset(this.getImageInfo());
    }

    _loadTextures(renderer)
    {
        return renderer.loadTextureForImageInfo(this.getImageInfo());
    }

    _releaseTextures(renderer)
    {
        renderer.releaseTextureForImageInfo(this.getImageInfo());
    }
};
```

**Методы `ISDKObjectTypeBase`:**

| Метод | Описание |
|---|---|
| `this.getImageInfo()` | Информация об изображении |
| `this.runtime` | Рантайм |

### 7.8 Actions / Conditions / Expressions в рантайме

```js
// actions.js
C3.Plugins.MyPlugin.Acts = {
    SetSpeed(speed) {
        // this — экземпляр плагина (SDKInstanceClass)
        this._speed = speed;
    },
    async DoSomethingAsync() { /* async action */ }
};

// conditions.js
C3.Plugins.MyPlugin.Cnds = {
    IsLargeNumber(num) {
        return num > 100;
    },
    OnHitSolid() {
        return true;  // триггеры всегда возвращают true
    }
};

// expressions.js
C3.Plugins.MyPlugin.Exps = {
    Speed() {
        return this._speed;   // number или string
    }
};
```

**Для behavior** — используются `C3.Behaviors.MyBehavior.Acts`, `.Cnds`, `.Exps`.

**Вызов сравнения:**
```js
C3.compare(this.value, cmpType, compareValue)
// cmpType — число из параметра типа "cmp"
```

**Событие C3:**
```js
new C3.Event("event-name")
new C3.Event("event-name", true)  // bubbles=true
```

### 7.9 Сохранение/загрузка состояния

Методы `_saveToJson()` и `_loadFromJson(o)` вызываются автоматически при сохранении и загрузке игры (savegames). Используйте **minify-proof синтаксис** для строковых ключей:

```js
_saveToJson()
{
    return {
        "speed": this._speed,   // строковые ключи — обязательно
        "enabled": this._enabled
    };
}

_loadFromJson(o)
{
    this._speed   = o["speed"];
    this._enabled = o["enabled"];
}
```

---

## 8. DOM-side Scripting

Используется когда плагин работает в **Web Worker** режиме и нуждается в доступе к DOM.

### 8.1 DOMElementHandler (для DOM-плагинов с элементами)

`c3runtime/domSide.js`:

```js
"use strict";

{
    const DOM_COMPONENT_ID = "mycompany-mydomplugin"; // должен совпадать с instance.js и plugin.js

    const HANDLER_CLASS = class MyDOMHandler extends globalThis.DOMElementHandler
    {
        constructor(iRuntime)
        {
            super(iRuntime, DOM_COMPONENT_ID);
        }

        // Вызывается при this._createElement() из instance.js
        CreateElement(elementId, state)
        {
            const elem = document.createElement("button");
            elem.style.position = "absolute";
            // ... настройка стилей ...

            elem.addEventListener("click", () => {
                this.PostToRuntimeElement("click", elementId /*, optionalData */);
            });

            this.UpdateState(elem, state);
            return elem;
        }

        // Вызывается при this._updateElementState() из instance.js
        UpdateState(elem, state)
        {
            elem.textContent = state["text"];
            // runtime автоматически управляет позицией, размером и видимостью элемента
        }
    };

    globalThis.RuntimeInterface.AddDOMHandlerClass(HANDLER_CLASS);
}
```

**Методы `DOMElementHandler`:**

| Метод | Описание |
|---|---|
| `this.PostToRuntimeElement(msgId, elementId, data?)` | Отправить сообщение конкретному экземпляру в рантайм |

**Runtime автоматически:**
- Позиционирует DOM-элемент в соответствии с позицией объекта в макете
- Управляет видимостью элемента
- Управляет z-index'ом

### 8.2 DOMHandler (для DOM-messaging без элементов)

`c3runtime/domSide.js`:

```js
"use strict";

{
    const DOM_COMPONENT_ID = "MyCompany_DOMMessaging";

    const HANDLER_CLASS = class MyDOMHandler extends globalThis.DOMHandler
    {
        constructor(iRuntime)
        {
            super(iRuntime, DOM_COMPONENT_ID);

            this.AddRuntimeMessageHandlers([
                ["get-initial-state", ()  => this._OnGetInitialState()],
                ["set-document-title", e => this._OnSetDocumentTitle(e)]
            ]);
        }

        _OnGetInitialState()
        {
            // Возвращаемое значение → resolves PostToDOMAsync() на runtime-side
            return { "document-title": document.title };
        }

        _OnSetDocumentTitle(e)
        {
            document.title = e["title"];
        }
    };

    globalThis.RuntimeInterface.AddDOMHandlerClass(HANDLER_CLASS);
}
```

**Методы `DOMHandler`:**

| Метод | Описание |
|---|---|
| `this.AddRuntimeMessageHandlers([ [msgId, fn], ... ])` | Зарегистрировать обработчики сообщений от рантайма |

**Со стороны рантайма (instance.js):**
```js
// Конструктор instance — без элемента, но с domComponentId:
super({ domComponentId: DOM_COMPONENT_ID });

// Отправить без ожидания:
this._postToDOM("set-document-title", { "title": "New Title" });

// Отправить и ждать ответа:
const result = await this._postToDOMAsync("get-initial-state");
```

---

## 9. Wrapper Extensions

Позволяют вызывать C++ DLL из рантайма.

### 9.1 Регистрация файлов (editor plugin.js)

```js
this._info.AddFileDependency({
    filename: "MyExtension_x86.ext.dll",
    type: "wrapper-extension",
    platform: "windows-x86"
});
this._info.AddFileDependency({
    filename: "MyExtension_x64.ext.dll",
    type: "wrapper-extension",
    platform: "windows-x64"
});
this._info.AddFileDependency({
    filename: "my_extension_x64.ext.so",
    type: "wrapper-extension",
    platform: "linux-x64"
});
// Опционально: "windows-arm64"
```

### 9.2 Runtime instance.js (wrapper extension)

```js
constructor()
{
    super({ wrapperComponentId: "my-extension" }); // должен совпадать с RegisterComponentId() в C++

    this._isWrapperExtAvailable = this._isWrapperExtensionAvailable();

    // Получать входящие сообщения от DLL:
    // this._addWrapperExtensionMessageHandler("msg-id", e => this._onMyMessage(e));

    if (this._isWrapperExtAvailable)
    {
        this.runtime.sdk.addLoadPromise(this._init());
    }
}

async _init()
{
    const result = await this._sendWrapperExtensionMessageAsync("init");
    const data = result;
    this._value = data["myValue"];
}
```

**Методы wrapper extension:**

| Метод | Описание |
|---|---|
| `this._isWrapperExtensionAvailable()` | Проверить загружен ли DLL |
| `this._sendWrapperExtensionMessageAsync(msgId, data?)` | Отправить асинхронное сообщение в DLL, ожидает `SendAsyncResponse()` на C++ стороне |
| `this._addWrapperExtensionMessageHandler(msgId, fn)` | Получать push-сообщения от DLL |

---

## 10. Effect SDK

### addon.json

```json
{
    "$schema": "../effect.addon.schema.json",
    "is-c3-addon": true,
    "type": "effect",
    "name": "My Effect",
    "id": "MyCompany_MyEffect",
    "supported-renderers": ["webgl", "webgpu"],
    "version": "1.0.0.0",
    "author": "Author",
    "website": "https://example.com",
    "documentation": "https://example.com",
    "description": "Short description.",
    "file-list": ["lang/en-US.json", "addon.json", "effect.fx", "effect.wgsl"],
    "category": "color",
    "blends-background": false,
    "cross-sampling": false,
    "preserves-opaqueness": true,
    "animated": false,
    "extend-box": { "horizontal": 0, "vertical": 0 },
    "parameters": [
        {
            "id": "color",
            "type": "color",
            "initial-value": [1, 0, 0],
            "uniform": "setColor"
        }
    ]
}
```

**Поля `parameters`:**

| Поле | Описание |
|---|---|
| `id` | Идентификатор параметра |
| `type` | `"color"`, `"float"`, `"percent"`, `"image"` |
| `initial-value` | Начальное значение |
| `uniform` | Имя uniform-переменной в шейдере |

### Шейдеры

- `effect.fx` — GLSL для WebGL
- `effect.wgsl` — WGSL для WebGPU
- `effect.webgl2.fx` — GLSL специально для WebGL 2 (опционально, иначе используется `effect.fx`)

---

## 11. Theme SDK

### addon.json

```json
{
    "$schema": "../theme.addon.schema.json",
    "is-c3-addon": true,
    "type": "theme",
    "name": "My Theme",
    "id": "MyTheme",
    "version": "1.0.0.0",
    "author": "Author",
    "website": "https://example.com",
    "documentation": "https://example.com",
    "description": "A custom editor theme.",
    "stylesheets": ["theme.css"],
    "file-list": ["lang/en-US.json", "addon.json", "icon.svg", "theme.css"]
}
```

Тема — только CSS (`theme.css`), без JavaScript. Переопределяет CSS-переменные редактора.

---

## 12. Looping Conditions (циклы в условиях)

Looping condition позволяет повторять тело блока событий для каждого элемента коллекции — аналог `For Each` в Construct.

### 12.1 aces.json — обязательные флаги

```json
{
    "id": "ForEachKey",
    "scriptName": "ForEachKey",
    "isLooping": true,
    "isTrigger": false,
    "params": []
}
```

**Критично:** `"isLooping": true` и `"isTrigger": false` должны стоять вместе.

### 12.2 Реализация в c3runtime (единственный правильный способ в SDK v2)

```js
function Cnd_ForEachKey()
{
    // 1. Создать контекст цикла
    const loopCtx = this.runtime.sdk.createLoopingConditionContext();

    // 2. Итерировать по данным
    for (const key of this._data.keys())
    {
        // 3. Проверить, не остановил ли пользователь цикл
        if (loopCtx.isStopped) break;

        // 4. Установить "текущий элемент" на instance, чтобы выражения его видели
        this._currentKey = key;

        // 5. Запустить тело блока событий для этой итерации
        loopCtx.retrigger();
    }

    // 6. Сбросить состояние и освободить контекст
    this._currentKey = "";
    loopCtx.release();

    // 7. Сама функция условия ВСЕГДА возвращает false
    //    Реальное выполнение происходит через loopCtx.retrigger()
    return false;
}
```

### 12.3 API объекта loopCtx

| Поле/Метод | Тип | Описание |
|---|---|---|
| `loopCtx.isStopped` | `boolean` | `true` если пользователь поставил «Stop loop» в событиях — нужно проверять на каждой итерации |
| `loopCtx.retrigger()` | `void` | Выполняет тело блока событий для текущей итерации |
| `loopCtx.release()` | `void` | **Обязательно вызывать** после завершения цикла |

### 12.4 Expressions для доступа к текущему элементу

Выражения просто читают поле, которое условие установило перед `retrigger()`:

```js
// expressions.js
Exp_CurrentKey()   { return this._currentKey; }
Exp_CurrentValue() { return this._data.get(this._currentKey) ?? 0; }
```

### 12.5 Полный шаблон «For Each» для любой коллекции

**aces.json:**
```json
{
    "id": "for-each-item",
    "scriptName": "ForEachItem",
    "isLooping": true,
    "isTrigger": false,
    "params": []
}
```

**c3runtime/conditions.js:**
```js
C3.Plugins.MyPlugin.Cnds = {
    ForEachItem()
    {
        const loopCtx = this.runtime.sdk.createLoopingConditionContext();

        for (const [key, value] of this._items)
        {
            if (loopCtx.isStopped) break;
            this._loopKey   = key;
            this._loopValue = value;
            loopCtx.retrigger();
        }

        this._loopKey   = "";
        this._loopValue = null;
        loopCtx.release();
        return false;
    }
};
```

**c3runtime/expressions.js:**
```js
C3.Plugins.MyPlugin.Exps = {
    LoopKey()   { return this._loopKey; },
    LoopValue() { return this._loopValue ?? 0; }
};
```

### 12.6 Что НЕЛЬЗЯ делать в looping condition

```js
// ❌ НЕВЕРНО — SDK v1 API, в SDK v2 не существуют:
this.runtime.GetEventSheetManager()
this.runtime.GetEventStack()
this.runtime.PushCopySol()
this.runtime.PopSol()

// ❌ НЕВЕРНО — возвращать true из looping condition:
return true;  // тело событий никогда не выполнится правильно

// ❌ НЕВЕРНО — забыть loopCtx.release():
loopCtx.retrigger();
return false;  // утечка ресурсов

// ✅ ВЕРНО — всегда полный цикл с проверкой:
const loopCtx = this.runtime.sdk.createLoopingConditionContext();
for (...) {
    if (loopCtx.isStopped) break;
    /* установить текущий элемент */
    loopCtx.retrigger();
}
/* сбросить текущий элемент */
loopCtx.release();
return false;
```

---

## 13. Утилиты C3

`self.C3` (или `globalThis.C3`) предоставляет ряд встроенных хелперов.

### Сравнение (для параметра типа `"cmp"`)

```js
// Параметр типа "cmp" приходит как число 0–5
self.C3.compare(leftValue, cmpOperator, rightValue)
// Возвращает boolean. Операторы: 0=equal, 1=not_equal, 2=less, 3=less_equal, 4=greater, 5=greater_equal
```

### Конвертация Map ↔ Object (для сохранения)

```js
// Map → plain object (для _saveToJson)
const obj = self.C3.MapToObject(this._data);

// plain object → Map (для _loadFromJson)
self.C3.ObjectToMap(obj, this._data);
```

### Событие

```js
new C3.Event("event-name")           // не всплывает
new C3.Event("event-name", true)     // bubbles = true
```

---

## 14. Частые ошибки ИИ-агентов

### ❌ НЕ существующие методы

```js
// НЕВЕРНО — таких методов нет:
this.GetX()          // нет, используйте this.x (runtime) или this._inst.GetX() (editor)
this.GetY()
this._info.SetCategory("general")  // ВЕРНО, но только в конструкторе plugin.js
C3.Plugins.MyPlugin.Initialize()  // не существует
this.CreateInstance()              // не существует на instance
SDK.Runtime                        // не существует в editor side
this.PostMessage()                 // не существует на instance

// НЕВЕРНО — неправильный регистр методов рантайма:
renderer.SetTexture()   // в рантайме: renderer.setTexture() (lowercase)
renderer.Quad3()        // в рантайме: renderer.quad3() (lowercase)
// В редакторе (SDK editor side) — PascalCase: iRenderer.SetTexture(), iRenderer.Quad3()
```

### ❌ Смешивание editor-side и runtime-side API

```js
// НЕВЕРНО в c3runtime/:
const SDK = globalThis.SDK;           // SDK доступен только в editor-файлах
SDK.Plugins.MyPlugin.Acts = { ... }   // НЕВЕРНО: в c3runtime нет SDK

// ВЕРНО в c3runtime/:
const C3 = globalThis.C3;
C3.Plugins.MyPlugin.Acts = { ... }
```

### ❌ Несовпадение ID

```js
// DOM_COMPONENT_ID должен быть ОДИНАКОВЫМ в:
// - c3runtime/plugin.js (super({ domComponentId: ... }))
// - c3runtime/instance.js (super({ domComponentId: ... }))
// - c3runtime/domSide.js (super(iRuntime, DOM_COMPONENT_ID))
```

### ❌ Неправильный вызов _trigger

```js
// НЕВЕРНО:
this._trigger("OnHitSolid");

// ВЕРНО:
this._trigger(C3.Behaviors.MyBehavior.Cnds.OnHitSolid);
this._trigger(C3.Plugins.MyPlugin.Cnds.OnClick);
```

### ❌ Забытый super._release()

```js
_release()
{
    // ОБЯЗАТЕЛЬНО вызывать super:
    super._release();
}
```

### ❌ Неправильная структура aces.json

```json
// НЕВЕРНО — объект верхнего уровня должен быть категориями, не массивом:
{ "actions": [...], "conditions": [...] }

// ВЕРНО:
{
    "my-category": {
        "actions": [...],
        "conditions": [...],
        "expressions": [...]
    }
}
```

### ❌ Передача ключа `scriptName` в expressions вместо `expressionName`

```json
// НЕВЕРНО:
{ "id": "speed", "scriptName": "Speed", "returnType": "number" }

// ВЕРНО:
{ "id": "speed", "expressionName": "Speed", "returnType": "number" }
```

### ✅ Правильное именование файлов behavior

```
behavior.js    ← не plugin.js
type.js
instance.js
c3runtime/behavior.js   ← не plugin.js
```

### ✅ Регистрация классов

```js
// Plugin:
PLUGIN_CLASS.Register(PLUGIN_ID, PLUGIN_CLASS);
// Behavior:
BEHAVIOR_CLASS.Register(BEHAVIOR_ID, BEHAVIOR_CLASS);

// Runtime — НЕ нужна регистрация, просто присвоение:
C3.Plugins.MyPlugin.Instance = MyInstance;
C3.Plugins.MyPlugin.Acts = { ... };
```

---

## Версионирование

| Поле | Формат |
|---|---|
| `version` в addon.json | `"MAJOR.MINOR.PATCH.REVISION"`, например `"1.0.0.0"` |
| `sdk-version` | всегда `2` |
| `min-construct-version` | строка, например `"r401"` |

---

## Быстрый старт

1. Скопируйте нужный sample из SDK (например `singleGlobalPlugin`)
2. Переименуйте `PLUGIN_ID` — **никогда не меняйте его после релиза**
3. Обновите `addon.json`: `id`, `name`, `version`
4. Обновите `lang/en-US.json` (ключ — `PLUGIN_ID.toLowerCase()`)
5. Добавьте ACE в `aces.json` и реализуйте в `c3runtime/actions.js`, `conditions.js`, `expressions.js`
6. Для TypeScript — редактируйте `.ts` файлы, `.js` файлы генерируются компилятором
