# BrowserWindow

> Создавайте окна браузера и управляйте ими.

Процесс: [Основной](../glossary.md#main-process)

```javascript
// В основном процессе.
const { BrowserWindow } = require('electron')

const win = new BrowserWindow({ width: 800, height: 600 })

// Загрузка удаленного URL
win.loadURL('https://github.com')

// Или загрузка локального HTML файла
win.loadFile('index.html')
```

## Окно без рамки

Для создания окна без хрома или прозрачного окна произвольной формы, можно использовать API [окна без рамки](frameless-window.md).

## Изящный показ окон

Когда страница загружается в окно напрямую, пользователь может увидеть постепенную загрузку страницы, что не является хорошей практикой для нативных приложений. Чтобы заставить окно показываться без визуальных скачков, есть два решения для разных ситуаций.

## Использование `ready-to-show` события

При загрузке страницы, после отрисовки страницы будет происходить событие `ready-to-show`, которое будет происходить первый раз, если окно до этого еще не было показано. Окно, показанное после этого события, не будет иметь визуальной ступенчатой подгрузки:

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ show: false })
win.once('ready-to-show', () => {
  win.show()
})
```

Обычно это событие происходит после события `did-finish-load`. Однако, страницы, включающие в себя удаленные ресурсы, могут продолжать подгружаться после происхождения события `did-finish-load`.

Пожалуйста, обратите внимание, что использование этого события означает, что рендерер будет считаться "видимым" и отрисуется, даже если `show` является false.  Это событие никогда не сработает, если вы используете `paintWhenInitiallyHidden: false`

## Настройка `backgroundColor`

Для больших приложений событие `ready-to-show` может вызываться слишком поздно, что может замедлить приложение. В этом случае рекомендуется показать окно немедленно, и использовать `backgroundColor`, задающий цвет фона Вашего приложения:

```javascript
const { BrowserWindow } = require('electron')

const win = new BrowserWindow({ backgroundColor: '#2e2c29' })
win.loadURL('https://github.com')
```

Обратите внимание, что даже для приложений, использующих `ready-to-show` события, по-прежнему рекомендуется установить `backgroundColor`, чтобы сделать приложение более нативным.

## Родительские и дочерние окна

С помощью параметра `parent`, Вы можете создавать дочерние окна:

```javascript
const { BrowserWindow } = require('electron')

const top = new BrowserWindow()
const child = new BrowserWindow({ parent: top })
child.show()
top.show()
```

Окно `child` будет всегда показано поверх окна `top`.

## Модальные окна

Модальное окно - дочернее окно, которое делает недоступным родительское окно. Чтобы создать модальное окно, Вы должны установить два параметра `parent` и `modal`:

```javascript
const { BrowserWindow } = require('electron')

const child = new BrowserWindow({ parent: top, modal: true, show: false })
child.loadURL('https://github.com')
child.once('ready-to-show', () => {
  child.show()
})
```

## Видимость страниц

[API видимости страниц][page-visibility-api] работает следующим образом:

* На всех платформах состояние видимости отслеживает скрыто/уменьшено окно или нет.
* Кроме того, на macOS, состояние видимости также отслеживает состояние перекрытия окна. Если окно перекрыто (т.е. полностью покрыто) другим окном, состояние видимости будет `hidden`. На других платформах состояние видимости будет `hidden`, только когда окно уменьшено или явно скрыто при помощи `win.hide()`.
* Если `BrowserWindow` создано с `show: false`, первоначальное состояние видимости будет `visible`, несмотря на фактически скрытое окно.
* Если `backgroundThrottling` отключено, состояние видимости останется `visible`, даже если окно уменьшено, закрыто или скрыто.

Рекомендуется приостановить дорогостоящие операции, когда состояние видимости `hidden`, для того чтобы свести к минимуму потребление энергии.

## Платформа заметок

* На macOS модальные окна будут отображены в виде страниц, прикрепленных к родительскому окну.
* На macOS дочерние окна будут находиться относительно родительского окна, во время передвижения родительского окна, тем временем на Windows и Linux дочерние окна не будут двигаться.
* На Linux тип модального окна будет поменян в `dialog`.
* На Linux многие среды рабочего стола не поддерживают скрытие модального окна.

## Класс: BrowserWindow

> Создавайте окна браузера и управляйте ими.

Процесс: [Основной](../glossary.md#main-process)

`BrowserWindow` это[EventEmitter][event-emitter].

Так создается новый экземпляр `BrowserWindow` с нативными свойствами, установленными в `options`.

### `new BrowserWindow([options])`

* `options` Object (опционально)
  * `width` Integer (опционально) - ширина окна в пикселях. По умолчанию `800`.
  * `height` Integer (опционально) - высота окна в пикселях. По умолчанию `600`.
  * `x` Integer (опционально) - (**обязателен**, если используется y) отступ окна слева от экрана. Значение по умолчанию центрирует окно.
  * `y` Integer (опционально) - (**обязателен**, если используется x) отступ окна сверху от экрана. Значение по умолчанию центрирует окно.
  * `useContentSize` Boolean (опционально) - `width` и `height` могут использоваться как размеры веб-страницы, это значит, что актуальный размер окна будет включать размер фрейма и будет немного крупнее. По умолчанию - `false`.
  * `center` Boolean (опционально) - показывает окно в центре экрана.
  * `minWidth` Integer (опционально) - минимальная ширина окна. По умолчанию `0`.
  * `minHeight` Integer (опционально) - минимальная высота окна. По умолчанию `0`.
  * `maxWidth` Integer (опционально) - максимальная ширина окна. По умолчанию нет ограничения.
  * `maxHeight` Integer (опционально) - максимальная высота окна. По умолчанию нет ограничения.
  * `resizable` Boolean (опционально) - будет ли окно изменять размеры. По умолчанию - `true`.
  * `movable` Boolean (optional) - можно ли будет перемещать окно. Не реализовано в Linux. По умолчанию - `true`.
  * `minimizable` Boolean (optional) - Whether window is minimizable. Не реализовано в Linux. По умолчанию - `true`.
  * `maximizable` Boolean (optional) - Whether window is maximizable. Не реализовано в Linux. По умолчанию - `true`.
  * `closable` Boolean (optional) - Whether window is closable. Не реализовано в Linux. По умолчанию - `true`.
  * `focusable` Boolean (опционально) - может ли быть окно в фокусе. По умолчанию - `true`. На Windows настройка `focusable: false` также подразумевает настройку `skipTaskbar: true`. На Linux настройка `focusable: false` прекращает взаимодействие окна с оконным менеджером, на Windows же всегда остается поверх всех рабочих областей.
  * `alwaysOnTop` Boolean (опционально) - будет ли окно всегда оставаться поверх других окон. По умолчанию - `false`.
  * `fullscreen` Boolean (опционально) - будет ли окно показываться во весь экран. Когда явно установлено `false`, на macOS кнопка полноэкранного режима будет скрыта или отключена. По умолчанию - `false`.
  * `fullscreenable` Boolean (опционально) - может ли окно быть в полноэкранном режиме. На macOS также кнопка увеличить/зумировать должна переключить в полноэкранный режим или увеличить окно. По умолчанию - `true`.
  * `simpleFullscreen` Boolean (optional) - Use pre-Lion fullscreen on macOS. По умолчанию - `false`.
  * `skipTaskbar` Boolean (optional) - Whether to show the window in taskbar. По умолчанию - `false`.
  * `kiosk` Boolean (optional) - Whether the window is in kiosk mode. По умолчанию - `false`.
  * `title` String (опционально) - заголовок окна по умолчанию. По умолчанию `"Electron"`. Если HTML-тег `<title>` определен в HTML-файле, загруженном с помощью `loadURL()`, то это свойство будет игнорироваться.
  * `icon` ([NativeImage](native-image.md) | String) (опционально) - иконка окна. На Windows рекомендуется использовать иконки `ICO`, чтобы получить лучший визуальный эффект, Вы также можете оставить неопределенным, чтобы был использован значок исполняемого файла.
  * `show` Boolean (необязательно) - Будет ли показано окно, когда будет создано. По умолчанию - `true`.
  * `paintWhenInitiallyHidden` Boolean (опционально) - Должен ли рендерер быть активным, когда `show` равен `false` и он только что создан.  Для `document.visibilityState` для корректной работы при первой загрузке с `show: false` необходимо установить значение `false`.  Установка этого в `false` приведёт к тому, что события `ready-to-show` не будут запускаться.  По умолчанию - `true`.
  * `frame` Boolean (optional) - Specify `false` to create a [Frameless Window](frameless-window.md). По умолчанию - `true`.
  * `parent` BrowserWindow (optional) - Specify parent window. По умолчанию - `null`.
  * `modal` Boolean (optional) - Whether this is a modal window. This only works when the window is a child window. По умолчанию - `false`.
  * `acceptFirstMouse` Boolean (optional) - Whether clicking an inactive window will also click through to the web contents. Default is `false` on macOS. This option is not configurable on other platforms.
  * `disableAutoHideCursor` Boolean (optional) - Whether to hide cursor when typing. По умолчанию - `false`.
  * `autoHideMenuBar` Boolean (optional) - Auto hide the menu bar unless the `Alt` key is pressed. По умолчанию - `false`.
  * `enableLargerThanScreen` Boolean (опционально) - позволяет окну изменять размер больше, чем экран. Относится только к macOS, так как другие ОС по умолчанию разрешают окна больше экрана. По умолчанию - `false`.
  * `backgroundColor` String (опционально) - фоновый цвет окна в HEX-формате, например `#66CD00`, `#FFF` или `#80FFFFFF` (альфа в формате #AARRGGBB поддерживается, если `transparent` установлено `true`). По умолчанию - `#FFF` (белый).
  * `hasShadow` Boolean (optional) - Whether window should have a shadow. По умолчанию - `true`.
  * `opacity` Number (optional) - Set the initial opacity of the window, between 0.0 (fully transparent) and 1.0 (fully opaque). This is only implemented on Windows and macOS.
  * `darkTheme` Boolean (optional) - Forces using dark theme for the window, only works on some GTK+3 desktop environments. По умолчанию - `false`.
  * `transparent` Boolean (опционально) - делает окно [прозрачным](frameless-window.md#transparent-window). По умолчанию - `false`. В Windows не работает до тех пор, пока окно не будет без фрейма.
  * `type` String (optional) - The type of window, default is normal window. See more about this below.
  * `visualEffectState` String (optional) - Specify how the material appearance should reflect window activity state on macOS. Must be used with the `vibrancy` property. Возможные значения:
    * `followWindow` - The backdrop should automatically appear active when the window is active, and inactive when it is not. This is the default.
    * `active` - The backdrop should always appear active.
    * `inactive` - The backdrop should always appear inactive.
  * `titleBarStyle` Строка (необязательно) _macOS_ _Windows_ - Стиль строки заголовка окна. По умолчанию - `default`. Возможные значения:
    * `по умолчанию` - результаты в стандартной строке заголовка для macOS или Windows, соответственно.
    * `скрытые` - результаты в скрытой строке заголовка и окно полного размера. На macOS в левом верхнем углу окна все еще есть стандартное управление окном ("светофоры"). На Windows, когда в сочетании с заголовком `titleBarOverlay: true` он активирует Overlay Window Controls Overlay (см. `titleBarOverlay` для получения дополнительной информации), иначе панель управления окном не будет показана.
    * `hiddenInset` - Только в macOS приводит к скрытой строке заголовка с альтернативным видом где кнопки светофора немного дальше от края окна.
    * `customButtonsOnHover` - Только в macOS приводит к скрытой строке заголовка и полному размеру окно содержимого, кнопки светофора будут отображаться при наведении курсора в верхнем левом углу окна.  **Примечание:** Эта опция в настоящее время экспериментальная.
  * `trafficLightPosition` [Point](structures/point.md) (optional) - Set a custom position for the traffic light buttons in frameless windows.
  * `roundedCorners` Boolean (optional) - Whether frameless window should have rounded corners on macOS. По умолчанию - `true`.
  * `fullscreenWindowTitle` Boolean (optional) _Deprecated_ - Shows the title in the title bar in full screen mode on macOS for `hiddenInset` titleBarStyle. По умолчанию - `false`.
  * `thickFrame` Boolenan (необязательно) - Использовать стиль `WS_THICKFRAME` на окнах с отсутствием рамок на Windows, добавляющий стандартные рамки окна. Установив значение `false`, тень окна и анимация окна будут удалены. По умолчанию - `true`.
  * `vibrancy` String (опционально) - добавить тип эффекта вибрации к окну, только на macOS. Может быть `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light`, `ultra-dark`, `header`, `sheet`, `window`, `hud`, `fullscreen-ui`, `tooltip`, `content`, `under-window` или `under-page`. Please note that `appearance-based`, `light`, `dark`, `medium-light`, and `ultra-dark` are deprecated and have been removed in macOS Catalina (10.15).
  * `zoomToPageWidth` Boolean (optional) - Controls the behavior on macOS when option-clicking the green stoplight button on the toolbar or by clicking the Window > Zoom menu item. Если `true`, окно будет увеличиваться до предпочтительной ширины веб-страницы при увеличении, `false` приведет к увеличению масштаба до ширины экрана. Это также повлияет на поведение при вызове `maximize()` напрямую. По умолчанию - `false`.
  * `tabbingIdentifier` String (опционально) - название группы вкладок, позволяет открывать окно как нативную вкладку в macOS 10.12+. Окна с одинаковым идентификатором вкладки будут сгруппированы вместе. Это также добавляет новую нативную кнопку вкладки в панель вкладок вашего окна и позволяет Вашему `приложению` и окну получать событие `new-window-for-tab`.
  * `webPreferences` Object (optional) - Settings of web page's features.
    * `devTools` Boolean (опционально) - включает инструменты разработчика. Если значение `false`, нельзя будет использовать `BrowserWindow.webContents.openDevTools()`, чтобы открыть инструменты разработчика. По умолчанию - `true`.
    * `nodeIntegration` Boolean (optional) - Whether node integration is enabled. По умолчанию - `false`.
    * `nodeIntegrationInWorker` Boolean (опционально) - включает интеграцию NodeJS в веб-воркерах. По умолчанию - `false`. Больше об этом можно найти в [многопоточности](../tutorial/multithreading.md).
    * `nodeIntegrationInSubFrames` Boolean (опционально) - экспериментальная опция для включения поддержки NodeJS в подфреймах, таких как iframes и дочерних окнах. Все Ваши предварительные загрузки будут загружены для каждого iframe, Вы можете использовать `process.isMainFrame`, чтобы определить в главном фрейме Вы или нет.
    * `preload` String (опционально) - Определяет скрипт, который будет загружен до других скриптов загружаемых в странице. Этот скрипт будет всегда иметь доступ к узлам API в не зависимости включено или выключено объединение узлов. Значение должно быть абсолютным файловым путем к скрипту. Когда интеграция узлов отключена, сценарий предварительной загрузки может повторно вводить глобальные символы узла обратно в глобальную область. Посмотреть пример [здесь](context-bridge.md#exposing-node-global-symbols).
    * `sandbox` Boolean (опционально) - если установлено в true, то в окне будет запущена песочница рендеринга, что делает ее совместимой с Chromium на уровне операционной системы и отключает Node.js. Это не тоже самое, что параметр `nodeIntegration`, доступные АПИ для предзагрузки более ограничены. Узнать больше об этой опции можно [здесь](../tutorial/sandbox.md).
    * `session` [Сессия](session.md#class-session) (опционально) - отсылает сессию используемую на странице. Вместо передачи объекта Session напрямую, вы можете также выбрать использование `partition` опции вместо которой принимает строку раздела. Когда оба `session` и `partition` определены, `session` будет приоритентней. По умолчанию используется session по умолчанию.
    * `partition` String (опционально) - Устанавливает сеанс, используемый страницей в соответствии со строкой раздела сессии. Если `partition` начинается с `persist:`, страница будет использовать постоянную сессию доступная всем страницам в приложении с некоторыми `partition`. Если нет `persist:` префикса, страница будет использовать сеанс в памяти. При присваивании одинаковой `partition`, разные страницы могут иметь одинаковую сессию. По умолчанию используется session по умолчанию.
    * `zoomFactor` Number (optional) - The default zoom factor of the page, `3.0` represents `300%`. По умолчанию - `1.0`.
    * `javascript` Boolean (optional) - Enables JavaScript support. По умолчанию - `true`.
    * `webSecurity` Boolean (опционально) - Когда `false`, отключается политика same-origin (обычно используется при тестировании вебсайтов людьми), и устанавливается `allowRunningInsecureContent` в `true`, если параметр не был установлен пользователем. По умолчанию - `true`.
    * `allowRunningInsecureContent` Boolean (optional) - Allow an https page to run JavaScript, CSS or plugins from http URLs. По умолчанию - `false`.
    * `images` Boolean (optional) - Enables image support. По умолчанию - `true`.
    * `textAreasAreResizable` Boolean (optional) - Make TextArea elements resizable. Default is `true`.
    * `webgl` Boolean (optional) - Enables WebGL support. По умолчанию - `true`.
    * `plugins` Boolean (optional) - Whether plugins should be enabled. По умолчанию - `false`.
    * `experimentalFeatures` Boolean (optional) - Enables Chromium's experimental features. По умолчанию - `false`.
    * `scrollBounce` Boolean (optional) - Enables scroll bounce (rubber banding) effect on macOS. По умолчанию - `false`.
    * `enableBlinkFeatures` String (опционально) - список строк функций, разделенных `запятой`, которые нужно включить, например `CSSVariables,KeyboardEventKey`. Полный список поддерживаемых возможностей можно найти в файле [RuntimeEnabledFeatures.json5][runtime-enabled-features].
    * `disableBlinkFeatures` String (опционально) - Список функциональных возможностей для выключения, разделяются `','`, например `CSSVariables,KeyboardEventKey`. Полный список поддерживаемых возможностей можно найти в файле [RuntimeEnabledFeatures.json5][runtime-enabled-features].
    * `defaultFontFamily` Object (optional) - Sets the default font for the font-family.
      * `standard` String (опционально) - По умолчанию `Times New Roman`.
      * `serif` String (опционально) - По умолчанию `Times New Roman`.
      * `sansSerif` String (опционально) - По умолчанию `Arial`.
      * `monospace` String (опционально) - По умолчанию `Courier New`.
      * `cursive` String (optional) - Defaults to `Script`.
      * `fantasy` String (optional) - Defaults to `Impact`.
    * `defaultFontSize` Integer (optional) - Defaults to `16`.
    * `defaultMonospaceFontSize` Integer (optional) - Defaults to `13`.
    * `minimumFontSize` Integer (optional) - Defaults to `0`.
    * `defaultEncoding` String (optional) - Defaults to `ISO-8859-1`.
    * `backgroundThrottling` Boolean (optional) - Whether to throttle animations and timers when the page becomes background. This also affects the [Page Visibility API](#page-visibility). Defaults to `true`.
    * `offscreen` Boolean (optional) - Whether to enable offscreen rendering for the browser window. Defaults to `false`. See the [offscreen rendering tutorial](../tutorial/offscreen-rendering.md) for more details.
    * `contextIsolation` Boolean (optional) - Whether to run Electron APIs and the specified `preload` script in a separate JavaScript context. Defaults to `true`. The context that the `preload` script runs in will only have access to its own dedicated `document` and `window` globals, as well as its own set of JavaScript builtins (`Array`, `Object`, `JSON`, etc.), which are all invisible to the loaded content. The Electron API will only be available in the `preload` script and not the loaded page. This option should be used when loading potentially untrusted remote content to ensure the loaded content cannot tamper with the `preload` script and any Electron APIs being used.  This option uses the same technique used by [Chrome Content Scripts][chrome-content-scripts].  You can access this context in the dev tools by selecting the 'Electron Isolated Context' entry in the combo box at the top of the Console tab.
    * `nativeWindowOpen` Boolean (optional) - Whether to use native `window.open()`. Defaults to `false`. Дочерние окна всегда будут отключены для интеграции узлов, если `nodeIntegrationInSubFrames` будет true. **Примечание:** Значение по умолчанию будет изменено на `true` в Electron 15.
    * `webviewTag` Boolean (опционально) - включает [`<webview>`-тег](webview-tag.md). Defaults to `false`. **Примечание:** Cкрипт `предварительной загрузки`, настроенный для `<webview>`, будет иметь интеграцию NodeJS, когда будет запущен, так что Вы должны убедиться, что удаленный/непроверенный контент не может создавать тег `<webview>` с возможно вредоносным скриптом `предварительной загрузки`. Вы можете использовать событие `will-attach-webview` на [webContents](web-contents.md), чтобы снять скрипт `предварительной загрузки` и проверить или изменить начальные настройки `<webview>`.
    * `additionalArguments` String[] (optional) - A list of strings that will be appended to `process.argv` in the renderer process of this app.  Useful for passing small bits of data down to renderer process preload scripts.
    * `safeDialogs` Boolean (optional) - Whether to enable browser style consecutive dialog protection. По умолчанию - `false`.
    * `safeDialogsMessage` String (optional) - The message to display when consecutive dialog protection is triggered. If not defined the default message would be used, note that currently the default message is in English and not localized.
    * `disableDialogs` Boolean (optional) - Whether to disable dialogs completely. Overrides `safeDialogs`. По умолчанию - `false`.
    * `navigateOnDragDrop` Boolean (optional) - Whether dragging and dropping a file or link onto the page causes a navigation. По умолчанию - `false`.
    * `autoplayPolicy` String (опционально) - политика автовоспроизведения для применения к содержимому в окне, может быть `no-user-gesture-required`, `user-gesture-required` или `document-user-activation-required`. По умолчанию `no-user-gesture-required`.
    * `disableHtmlFullscreenWindowResize` Boolean (optional) - Whether to prevent the window from resizing when entering HTML Fullscreen. Default is `false`.
    * `accessibleTitle` String (optional) - An alternative title string provided only to accessibility tools such as screen readers. This string is not directly visible to users.
    * `spellcheck` Boolean (optional) - Whether to enable the builtin spellchecker. По умолчанию - `true`.
    * `enableWebSQL` Boolean (optional) - Whether to enable the [WebSQL api](https://www.w3.org/TR/webdatabase/). По умолчанию - `true`.
    * `v8CacheOptions` String (optional) - Enforces the v8 code caching policy used by blink. Accepted values are
      * `none` - Disables code caching
      * `code` - Heuristic based code caching
      * `bypassHeatCheck` - Bypass code caching heuristics but with lazy compilation
      * `bypassHeatCheckAndEagerCompile` - Same as above except compilation is eager. Default policy is `code`.
    * `enablePreferredSizeMode` Boolean (optional) - Whether to enable preferred size mode. The preferred size is the minimum size needed to contain the layout of the document—without requiring scrolling. Enabling this will cause the `preferred-size-changed` event to be emitted on the `WebContents` when the preferred size changes. По умолчанию - `false`.
  * `titleBarOverlay` [OverlayOptions](structures/overlay-options.md) | Boolean (optional) -  When using a frameless window in conjuction with `win.setWindowButtonVisibility(true)` on macOS or using a `titleBarStyle` so that the standard window controls ("traffic lights" on macOS) are visible, this property enables the Window Controls Overlay [JavaScript APIs][overlay-javascript-apis] and [CSS Environment Variables][overlay-css-env-vars]. Specifying `true` will result in an overlay with default system colors. По умолчанию - `false`.  On Windows, the [OverlayOptions](structures/overlay-options.md) can be used instead of a boolean to specify colors for the overlay.

Когда установлен минимальный или максимальный размер окна, при помощи `minWidth`/`maxWidth`/`minHeight`/`maxHeight`, это ограничивает только пользователей. Это не позволит Вам установить размер, который не будет следовать ограничениям размера, в `setBounds`/`setSize` или в конструкторе `BrowserWindow`.

The possible values and behaviors of the `type` option are platform dependent. Возможные значения:

* На Linux возможны типы `desktop`, `dock`, `toolbar`, `splash`, `notification`.
* On macOS, possible types are `desktop`, `textured`.
  * Тип `textured` добавляет вид металлического градиента (`NSTexturedBackgroundWindowMask`).
  * Тип `desktop` размещает окно на уровень фонового окна рабочего стола (`kCGDesktopWindowLevel - 1`). Обратите внимание, что окно рабочего стола не будет получить события фокуса, клавиатуры или мыши, но Вы можете использовать `globalShortcut`, чтобы получать ввод.
* На Windows возможен тип `toolbar`.

### События экземпляра

Объекты созданные с помощью `new BrowserWindow` имеют следующие события:

**Примечание:** Некоторые методы доступны только в определенных операционных системах и помечены как таковые.

#### Событие: 'page-title-updated'

Возвращает:

* Событие типа `event`
* `title` String
* `explicitSet` Boolean

Вызывается, когда документ меняет свой заголовок, вызов `event.preventDefault()` предотвратит изменение заголовка родного окна. `explicitSet` является false, когда заголовок синтезирован из url файла.

#### Событие: 'close'

Возвращает:

* Событие типа `event`

Происходит при закрытии окна. Оно происходит перед событиями `beforeunload` и `unload` в DOM. Вызов `event.preventDefault()` предотвратит закрытие.

Скорее всего, Вы захотите использовать обработчик `beforeunload`, чтобы решить, когда окно должно быть закрыто, который также будет вызываться, когда окно перезагружается. В Electron возврат любого значения, отличного от `undefined`, предотвратит закрытие. Например:

```javascript
window.onbeforeunload = (e) => {
  console.log('Я не хочу быть закрыт')

  // В отличие от браузеров, пользователю будет показано окно с сообщением.
  // Возврат любого значения незаметно отменит закрытие.
  // Рекомендуется использовать dialog API, чтобы дать пользователям
  // возможность подтвердить закрытие приложения.
  e.returnValue = false // идентично `return false`, но в использовании не рекомендуется
}
```

_**Примечание**: Существует тонкая разница между поведением `window.onbeforeunload = handler` и `window.addEventListener('beforeunload', handler)`. Рекомендуется всегда устанавливать `event.returnValue` явно, вместо того, чтобы просто возвращать значение, поскольку первое работает более последовательно в Electron._

#### Событие: 'closed'

происходит когда окно приложения закрыто. After you have received this event you should remove the reference to the window and avoid using it any more.

#### Событие: 'session-end' _Windows_

Происходит, когда сеанс окна заканчивается из-за выключения, перезагрузки компьютера или отключения сеанса.

#### Событие: 'unresponsive'

Вызывается, когда страница "не отвечает".

#### Событие: 'responsive'

Происходит, когда страница, которая "не отвечала", снова реагирует.

#### Событие: 'blur'

Вызывается, когда окно теряет фокус.

#### Событие: 'focus'

Вызывается, когда на окне фокусируются.

#### Событие: 'show'

Происходит, когда отображается окно.

#### Событие: 'hide'

Происходит, когда окно спрятано.

#### Событие: 'ready-to-show'

Происходит, когда веб-страница была отрисована (пока не отображена) и окно может быть отображено без визуального мерцания.

Пожалуйста, обратите внимание, что использование этого события означает, что рендерер будет считаться "видимым" и отрисуется, даже если `show` является false.  Это событие никогда не сработает, если вы используете `paintWhenInitiallyHidden: false`

#### Событие: 'maximize'

Происходит, когда окно увеличивается до предела.

#### Событие: 'unmaximize'

Происходит, когда окно выходит из увеличенного состояния.

#### Событие: 'minimize'

Происходит, когда окно было свернуто.

#### Событие: 'restore'

Происходит, когда окно восстанавливается из свернутого состояния.

#### Событие: 'will-resize' _macOS_ _Windows_

Возвращает:

* Событие типа `event`
* `newBounds` [Rectangle](structures/rectangle.md) - Размер окна, на который будет изменено.

Emitted before the window is resized. Calling `event.preventDefault()` will prevent the window from being resized.

Note that this is only emitted when the window is being resized manually. Resizing the window with `setBounds`/`setSize` will not emit this event.

#### Событие: 'resize'

Происходит после того, как изменился размер окна.

#### Event: 'resized' _macOS_ _Windows_

Emitted once when the window has finished being resized.

This is usually emitted when the window has been resized manually. On macOS, resizing the window with `setBounds`/`setSize` and setting the `animate` parameter to `true` will also emit this event once resizing has finished.

#### Событие: 'will-move'  _macOS_ _Windows_

Возвращает:

* Событие типа `event`
* `newBounds` [Rectangle](structures/rectangle.md) - Расположение, куда окно будет перемещено.

Emitted before the window is moved. On Windows, calling `event.preventDefault()` will prevent the window from being moved.

Note that this is only emitted when the window is being resized manually. Resizing the window with `setBounds`/`setSize` will not emit this event.

#### Событие: 'move'

Происходит, когда окно перемещено на новое место.

#### Event: 'moved' _macOS_ _Windows_

Происходит единожды, когда окно перемещается в новое положение.

__Примечание__: На macOS это событие является псевдонимом `move`.

#### Событие: 'enter-full-screen'

Происходит, когда окно переходит в полноэкранный режим.

#### Событие: 'leave-full-screen'

Происходит, когда окно выходит из полноэкранного режима.

#### Событие: 'enter-html-full-screen'

Происходит, когда окно входит в полноэкранный режим с помощью HTML API.

#### Событие: 'leave-html-full-screen'

Происходит, когда окно выходит из полноэкранного режима с помощью HTML API.

#### Событие: 'always-on-top-changed'

Возвращает:

* Событие типа `event`
* `isAlwaysOnTop` Boolean

Происходит, когда окно переключает режим отображения поверх всех окон.

#### Событие: 'app-command' _Windows_ _Linux_

Возвращает:

* Событие типа `event`
* `command` String

Происходит, когда вызывается [команда приложения](https://msdn.microsoft.com/en-us/library/windows/desktop/ms646275(v=vs.85).aspx). Обычно это касается клавиатурных медиа-клавиш или команд браузера, а также кнопки "Назад", встроенной в некоторые мыши на Windows.

Команды в нижнем регистре, подчеркивание заменено на дефисы, а префикс `APPCOMMAND_` обрезан. например `APPCOMMAND_BROWSER_BACKWARD` происходит как `browser-backward`.

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow()
win.on('app-command', (e, cmd) => {
  // Navigate the window back when the user hits their mouse back button
  if (cmd === 'browser-backward' && win.webContents.canGoBack()) {
    win.webContents.goBack()
  }
})
```

Следующие команды приложения явно поддерживаются на Linux:

* `browser-backward`
* `browser-forward`

#### Событие: 'scroll-touch-begin' _macOS_

Происходит, когда начинается прокрутка колеса.

#### Событие: 'scroll-touch-end' _macOS_

Происходит, когда заканчивается прокрутка колеса.

#### Событие: 'scroll-touch-edge' _macOS_

Происходит, когда прокрутка колеса достигает края элемента.

#### Событие: 'swipe' _macOS_

Возвращает:

* Событие типа `event`
* `direction` String

Emitted on 3-finger swipe. Possible directions are `up`, `right`, `down`, `left`.

Метод, лежащий в основе этого события, создан для обработки событий смахивания на устаревших трекпадах в стиле macOS, где содержимое на экране нельзя переместить смахнув. Большинство трекпадов macOS по умолчанию не настроены под этот вид смахивания, поэтому для того чтобы правильно работал 'Переход между страницами смахиванием' перейдите в `Системные настройки > Трекпад > Другие жесты ` и включите "Смахивание двумя или тремя пальцами".

#### Событие: 'rotate-gesture' _macOS_

Возвращает:

* Событие типа `event`
* `rotation` Float

Генерируется жестом вращения трекпада. Непрерывно генерируется до тех пор, пока жест поворота не будет завершен. Значение `rotation` при каждом выбросе - угол поворота в градусах, отсчитанный от последнего выброса. Последнее событие на жесте вращения всегда будет иметь значение `0`. Значения вращения против часовой стрелки положительные, по часовой стрелке отрицательные.

#### Событие: 'sheet-begin' _macOS_

Происходит, когда окно открывает лист.

#### Событие: 'sheet-end' _macOS_

Происходит, когда окно закрыло лист.

#### Событие: 'new-window-for-tab' _macOS_

Происходит, когда нажимается нативная кнопка новой вкладки.

#### Event: 'system-context-menu' _Windows_

Возвращает:

* Событие типа `event`
* `point` [Point](structures/point.md) - The screen coordinates the context menu was triggered at

Emitted when the system context menu is triggered on the window, this is normally only triggered when the user right clicks on the non-client area of your window.  This is the window titlebar or any area you have declared as `-webkit-app-region: drag` in a frameless window.

Calling `event.preventDefault()` will prevent the menu from being displayed.

### Статические методы

Класс `BrowserWindow` имеет следующие статические методы:

#### `BrowserWindow.getAllWindows()`

Возвращает `BrowserWindow[]` - массив всех открытых окон браузера.

#### `BrowserWindow.getFocusedWindow()`

Возвращает `BrowserWindow | null` - окно, которое сфокусировано в этом приложении, иначе возвращает `null`.

#### `BrowserWindow.fromWebContents(webContents)`

* `webContents` [WebContents](web-contents.md)

Возвращает `BrowserWindow | null` - окно, которое владеет объектом `webContents`, или имеет `null` если не содержит контент.

#### `BrowserWindow.fromBrowserView(browserView)`

* `browserView` [BrowserView](browser-view.md)

Возвращает `BrowserWindow` - окно, которое владеет объектом `webContents`. If the given view is not attached to any window, returns `null`.

#### `BrowserWindow.fromId(id)`

* `id` Integer

Возвращает `BrowserWindow | null` - окно с указанным `id`.

### Свойства экземпляра

Объекты, созданные с помощью `new BrowserWindow`, имеют следующие свойства:

```javascript
const { BrowserWindow } = require('electron')
// In this example `win` is our instance
const win = new BrowserWindow({ width: 800, height: 600 })
win.loadURL('https://github.com')
```

#### `win.webContents` _Только чтение_

A `WebContents` object this window owns. All web page related events and operations will be done via it.

Смотрите [документацию `webContents`](web-contents.md) для его методов и событий.

#### `win.id` _Только чтение_

Свойство `Integer` представляющее уникальный идентификатор окна. Each ID is unique among all `BrowserWindow` instances of the entire Electron application.

#### `win.autoHideMenuBar`

A `Boolean` property that determines whether the window menu bar should hide itself automatically. Once set, the menu bar will only show when users press the single `Alt` key.

Если панель меню уже видна, установка этого свойства в `true` не будет скрывать ее немедленно.

#### `win.simpleFullScreen`

A `Boolean` property that determines whether the window is in simple (pre-Lion) fullscreen mode.

#### `win.fullScreen`

A `Boolean` property that determines whether the window is in fullscreen mode.

#### `win.focusable` _Windows_ _macOS_

A `Boolean` property that determines whether the window is focusable.

#### `win.visibleOnAllWorkspaces`

A `Boolean` property that determines whether the window is visible on all workspaces.

**Note:** Always returns false on Windows.

#### `win.shadow`

A `Boolean` property that determines whether the window has a shadow.

#### `win.menuBarVisible` _Windows_ _Linux_

A `Boolean` property that determines whether the menu bar should be visible.

**Note:** If the menu bar is auto-hide, users can still bring up the menu bar by pressing the single `Alt` key.

#### `win.kiosk`

A `Boolean` property that determines whether the window is in kiosk mode.

#### `win.documentEdited` _macOS_

A `Boolean` property that specifies whether the window’s document has been edited.

The icon in title bar will become gray when set to `true`.

#### `win.representedFilename` _macOS_

A `String` property that determines the pathname of the file the window represents, and the icon of the file will show in window's title bar.

#### `win.title`

A `String` property that determines the title of the native window.

**Note:** The title of the web page can be different from the title of the native window.

#### `win.minimizable`

Свойство `Boolean` определяет, может ли окно быть вручную свернуто пользователем.

В Linux setter это невозможно, хотя getter возвращает `true`.

#### `win.maximizable`

Свойство `Boolean` определяет, может ли окно быть вручную развернуто пользователем.

В Linux setter это невозможно, хотя getter возвращает `true`.

#### `win.fullScreenable`

Свойство `Boolean` которое определяет, может ли кнопка увеличить/зумировать окно переключать полноэкранный режим или увеличивать до предела окно.

#### `win.resizable`

Свойство `Boolean` определяет, может ли окно быть вручную изменено пользователем.

#### `win.closable`

Свойство `Boolean` определяет, может ли окно быть вручную закрыто пользователем.

В Linux setter это невозможно, хотя getter возвращает `true`.

#### `win.movable`

Свойство `Boolean` определяет, может ли окно перемещаться пользователем.

В Linux setter это невозможно, хотя getter возвращает `true`.

#### `win.excludedFromShownWindowsMenu` _macOS_

A `Boolean` property that determines whether the window is excluded from the application’s Windows menu. По умолчанию `false`.

```js
const win = new BrowserWindow({ height: 600, width: 600 })

const template = [
  {
    role: 'windowmenu'
  }
]

win.excludedFromShownWindowsMenu = true

const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)
```

#### `win.accessibleTitle`

A `String` property that defines an alternative title provided only to accessibility tools such as screen readers. This string is not directly visible to users.

### Методы экземпляра

Objects created with `new BrowserWindow` have the following instance methods:

**Примечание:** Некоторые методы доступны только в определенных операционных системах и помечены как таковые.

#### `win.destroy()`

Force closing the window, the `unload` and `beforeunload` event won't be emitted for the web page, and `close` event will also not be emitted for this window, but it guarantees the `closed` event will be emitted.

#### `win.close()`

Try to close the window. This has the same effect as a user manually clicking the close button of the window. The web page may cancel the close though. See the [close event](#event-close).

#### `win.focus()`

Focuses on the window.

#### `win.blur()`

Removes focus from the window.

#### `win.isFocused()`

Returns `Boolean` - Whether the window is focused.

#### `win.isDestroyed()`

Returns `Boolean` - Whether the window is destroyed.

#### `win.show()`

Shows and gives focus to the window.

#### `win.showInactive()`

Shows the window but doesn't focus on it.

#### `win.hide()`

Скрывает окно.

#### `win.isVisible()`

Returns `Boolean` - Whether the window is visible to the user.

#### `win.isModal()`

Returns `Boolean` - Whether current window is a modal window.

#### `win.maximize()`

Maximizes the window. This will also show (but not focus) the window if it isn't being displayed already.

#### `win.unmaximize()`

Unmaximizes the window.

#### `win.isMaximized()`

Возвращает `Boolean` - если окно развернуто.

#### `win.minimize()`

Minimizes the window. On some platforms the minimized window will be shown in the Dock.

#### `win.restore()`

Restores the window from minimized state to its previous state.

#### `win.isMinimized()`

Возвращает `Boolean` - если окно свернуто.

#### `win.setFullScreen(flag)`

* `flag` Boolean

Sets whether the window should be in fullscreen mode.

#### `win.isFullScreen()`

Returns `Boolean` - Whether the window is in fullscreen mode.

#### `win.setSimpleFullScreen(flag)` _macOS_

* `flag` Boolean

Enters or leaves simple fullscreen mode.

Simple fullscreen mode emulates the native fullscreen behavior found in versions of macOS prior to Lion (10.7).

#### `win.isSimpleFullScreen()` _macOS_

Returns `Boolean` - Whether the window is in simple (pre-Lion) fullscreen mode.

#### `win.isNormal()`

Returns `Boolean` - Whether the window is in normal state (not maximized, not minimized, not in fullscreen mode).

#### `win.setAspectRatio(aspectRatio[, extraSize])`

* `aspectRatio` Float - The aspect ratio to maintain for some portion of the content view.
* `extraSize` [Size](structures/size.md) (optional) _macOS_ - The extra size not to be included while maintaining the aspect ratio.

This will make a window maintain an aspect ratio. The extra size allows a developer to have space, specified in pixels, not included within the aspect ratio calculations. This API already takes into account the difference between a window's size and its content size.

Consider a normal window with an HD video player and associated controls. Perhaps there are 15 pixels of controls on the left edge, 25 pixels of controls on the right edge and 50 pixels of controls below the player. In order to maintain a 16:9 aspect ratio (standard aspect ratio for HD @1920x1080) within the player itself we would call this function with arguments of 16/9 and
{ width: 40, height: 50 }. The second argument doesn't care where the extra width and height are within the content view--only that they exist. Sum any extra width and height areas you have within the overall content view.

The aspect ratio is not respected when window is resized programmingly with APIs like `win.setSize`.

#### `win.setBackgroundColor(backgroundColor)`

* `backgroundColor` String - Window's background color as a hexadecimal value, like `#66CD00` or `#FFF` or `#80FFFFFF` (alpha is supported if `transparent` is `true`). По умолчанию - `#FFF` (белый).

Sets the background color of the window. See [Setting `backgroundColor`](#setting-backgroundcolor).

#### `win.previewFile(path[, displayName])` _macOS_

* `path` String - абсолютный путь до файла, для предпросмотра в QuickLook. Это важно, так как QuickLook использует имя файла и расширение файла из пути, чтобы определить тип содержимого файла для открытия.
* `displayName` String (опционально) - имя файла, для отображения в модальном виде QuickLook. Это чисто визуально и не влияет на тип содержимого файла. По умолчанию `path`.

Использует [QuickLook][quick-look] для предпросмотра файла, по данному пути.

#### `win.closeFilePreview()` _macOS_

Закрывает текущую открытую панель [QuickLook][quick-look].

#### `win.setBounds(bounds[, animate])`

* `bounds` Partial<[Rectangle](structures/rectangle.md)>
* `animate` Boolean (необязательно) _macOS_

Resizes and moves the window to the supplied bounds. Any properties that are not supplied will default to their current values.

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow()

// Установить все свойства границы
win.setBounds({ x: 440, y: 225, width: 800, height: 600 })

// Установить одно свойство границы
win.setBounds({ width: 100 })

// { x: 440, y: 225, width: 100, height: 600 }
console.log(win.getBounds())
```

#### `win.getBounds()`

Возвращает [`Rectangle`](structures/rectangle.md) - `границы` окна как `объект`.

#### `win.getBackgroundColor()`

Returns `String` - Gets the background color of the window. See [Setting `backgroundColor`](#setting-backgroundcolor).

#### `win.setContentBounds(bounds[, animate])`

* `bounds` [Rectangle](structures/rectangle.md)
* `animate` Boolean (необязательно) _macOS_

Меняет размеры и перемещает клиентскую область окна (например, веб-страницу) на заданные границы.

#### `win.getContentBounds()`

Возвращает [`Rectangle`](structures/rectangle.md) - `границы` области клиентского окна как `объект`.

#### `win.getNormalBounds()`

Возвращает [`Rectangle`](structures/rectangle.md) - содержит границы окна в нормальном состоянии

**Примечание:** Независимо от текущего состояния окна: увеличено до предела, свернуто или в полноэкранном режиме, эта функция всегда возвратит позицию и размер окна в нормальном состоянии. В нормальном состоянии, getBounds и getNormalBounds возвращают тот же [`Rectangle`](structures/rectangle.md).

#### `win.setEnabled(enable)`

* `enable` Boolean

Включает или выключает окно.

#### `win.isEnabled()`

Returns `Boolean` - whether the window is enabled.

#### `win.setSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (необязательно) _macOS_

Resizes the window to `width` and `height`. If `width` or `height` are below any set minimum size constraints the window will snap to its minimum size.

#### `win.getSize()`

Возвращает `Integer[]` - Содержит высоту и ширину окна.

#### `win.setContentSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (необязательно) _macOS_

Меняет размер клиентской области окна (например, веб-страница) на `width` и `height`.

#### `win.getContentSize()`

Возвращает `Integer[]` - содержит ширину и высоту клиентской области окна.

#### `win.setMinimumSize(width, height)`

* `width` Integer
* `height` Integer

Устанавливает минимальный размер окна на `width` и `height`.

#### `win.getMinimumSize()`

Возвращает `Integer[]` - содержит минимальную ширину и высоту окна.

#### `win.setMaximumSize(width, height)`

* `width` Integer
* `height` Integer

Устанавливает максимальный размер окна на `width` и `height`.

#### `win.getMaximumSize()`

Возвращает `Integer[]` - содержит максимальную ширину и высоту окна.

#### `win.setResizable(resizable)`

* `resizable` Boolean

Sets whether the window can be manually resized by the user.

#### `win.isResizable()`

Returns `Boolean` - Whether the window can be manually resized by the user.

#### `win.setMovable(movable)` _macOS_ _Windows_

* `movable` Boolean

Sets whether the window can be moved by user. On Linux does nothing.

#### `win.isMovable()` _macOS_ _Windows_

Возвращает `Boolean` - может пользователь перемещать окно или нет.

На Linux всегда возвращает `true`.

#### `win.setMinimizable(minimizable)` _macOS_ _Windows_

* `minimizable` Boolean

Sets whether the window can be manually minimized by user. On Linux does nothing.

#### `win.isMinimizable()` _macOS_ _Windows_

Returns `Boolean` - Whether the window can be manually minimized by the user.

На Linux всегда возвращает `true`.

#### `win.setMaximizable(maximizable)` _macOS_ _Windows_

* `maximizable` Boolean

Sets whether the window can be manually maximized by user. On Linux does nothing.

#### `win.isMaximizable()` _macOS_ _Windows_

Возвращает `Boolean` - может пользователь вручную увеличивать до предела окно или нет.

На Linux всегда возвращает `true`.

#### `win.setFullScreenable(fullscreenable)`

* `fullscreenable` Boolean

Sets whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.isFullScreenable()`

Returns `Boolean` - Whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.setClosable(closable)` _macOS_ _Windows_

* `closable` Boolean

Sets whether the window can be manually closed by user. On Linux does nothing.

#### `win.isClosable()` _macOS_ _Windows_

Возвращает `Boolean` - может пользователь вручную закрывать окно или нет.

На Linux всегда возвращает `true`.

#### `win.setAlwaysOnTop(flag[, level][, relativeLevel])`

* `flag` Boolean
* `level` String (опционально) _macOS_ _Windows_ - Значения включают `normal`, `floating`, `torn-off-menu`, `modal-panel`, `main-menu`, `status`, `pop-up-menu`, `screen-saver` и ~~`dock`~~ (Устарело). По умолчанию `floating`, когда `flag` установлен true. `level` сбрасывается на `normal`, когда флаг устанавливается false. Обратите внимание, что от `floating` до `status` включительно, окно находится под Dock в macOS и под панелью задач в Windows. От `pop-up-menu` и выше отображается над Dock на macOS и выше панели задач на Windows. Смотрите [документацию macOS][window-levels] для подробностей.
* `relativeLevel` Integer (опционально) _macOS_ - количество слоев выше, чтобы установить окно относительно заданного `level`. По умолчанию - `0`. Обратите внимание, что Apple не рекомендует устанавливать уровни выше, чем 1 верхнего `screen-saver`.

Sets whether the window should show always on top of other windows. After setting this, the window is still a normal window, not a toolbox window which can not be focused on.

#### `win.isAlwaysOnTop()`

Возвращает `Boolean` - всегда ли окно поверх остальных окон.

#### `win.moveAbove(mediaSourceId)`

* `mediaSourceId` String - Window id in the format of DesktopCapturerSource's id. For example "window:1869:0".

Moves window above the source window in the sense of z-order. If the `mediaSourceId` is not of type window or if the window does not exist then this method throws an error.

#### `win.moveTop()`

Перемещает окно на верх(z-order) независимо от фокуса

#### `win.center()`

Перемещает окно в центр экрана.

#### `win.setPosition(x, y[, animate])`

* `x` Integer
* `y` Integer
* `animate` Boolean (необязательно) _macOS_

Перемещает окно на `x` и `y`.

#### `win.getPosition()`

Возвращает `Integer[]` - содержит текущую позицию окна.

#### `win.setTitle(title)`

* `title` String

Изменяет название нативного окна на `title`.

#### `win.getTitle()`

Возвращает `String` - название нативного окна.

**Примечание:** Название веб-страницы может отличаться от названия нативного окна.

#### `win.setSheetOffset(offsetY[, offsetX])` _macOS_

* `offsetY` Float
* `offsetX` Float (опционально)

Changes the attachment point for sheets on macOS. By default, sheets are attached just below the window frame, but you may want to display them beneath a HTML-rendered toolbar. Например:

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow()

const toolbarRect = document.getElementById('toolbar').getBoundingClientRect()
win.setSheetOffset(toolbarRect.height)
```

#### `win.flashFrame(flag)`

* `flag` Boolean

Starts or stops flashing the window to attract user's attention.

#### `win.setSkipTaskbar(skip)`

* `skip` Boolean

Makes the window not show in the taskbar.

#### `win.setKiosk(flag)`

* `flag` Boolean

Enters or leaves kiosk mode.

#### `win.isKiosk()`

Returns `Boolean` - Whether the window is in kiosk mode.

#### `win.isTabletMode()` _Windows_

Returns `Boolean` - Whether the window is in Windows 10 tablet mode.

Since Windows 10 users can [use their PC as tablet](https://support.microsoft.com/en-us/help/17210/windows-10-use-your-pc-like-a-tablet), under this mode apps can choose to optimize their UI for tablets, such as enlarging the titlebar and hiding titlebar buttons.

This API returns whether the window is in tablet mode, and the `resize` event can be be used to listen to changes to tablet mode.

#### `win.getMediaSourceId()`

Returns `String` - Window id in the format of DesktopCapturerSource's id. For example "window:1324:0".

More precisely the format is `window:id:other_id` where `id` is `HWND` on Windows, `CGWindowID` (`uint64_t`) on macOS and `Window` (`unsigned long`) on Linux. `other_id` is used to identify web contents (tabs) so within the same top level window.

#### `win.getNativeWindowHandle()`

Returns `Buffer` - The platform-specific handle of the window.

The native type of the handle is `HWND` on Windows, `NSView*` on macOS, and `Window` (`unsigned long`) on Linux.

#### `win.hookWindowMessage(message, callback)` _Windows_

* `message` Integer
* `callback` Function
  * `wParam` any - The `wParam` provided to the WndProc
  * `lParam` any - The `lParam` provided to the WndProc

Hooks a windows message. The `callback` is called when the message is received in the WndProc.

#### `win.isWindowMessageHooked(message)` _Windows_

* `message` Integer

Returns `Boolean` - `true` or `false` depending on whether the message is hooked.

#### `win.unhookWindowMessage(message)` _Windows_

* `message` Integer

Unhook the window message.

#### `win.unhookAllWindowMessages()` _Windows_

Unhooks all of the window messages.

#### `win.setRepresentedFilename(filename)` _macOS_

* `filename` String

Sets the pathname of the file the window represents, and the icon of the file will show in window's title bar.

#### `win.getRepresentedFilename()` _macOS_

Returns `String` - The pathname of the file the window represents.

#### `win.setDocumentEdited(edited)` _macOS_

* `edited` Boolean

Specifies whether the window’s document has been edited, and the icon in title bar will become gray when set to `true`.

#### `win.isDocumentEdited()` _macOS_

Returns `Boolean` - Whether the window's document has been edited.

#### `win.focusOnWebView()`

#### `win.blurWebView()`

#### `win.capturePage([rect])`

* `rect` [Rectangle](structures/rectangle.md) (optional) - The bounds to capture

Возвращает `Promise<NativeImage>` - разрешается с [NativeImage](native-image.md)

Захватывает снимок страницы в границах `rect`. Пропустив `rect`, будет сделан захват всей видимой страницы. If the page is not visible, `rect` may be empty.

#### `win.loadURL(url[, options])`

* `url` String
* `options` Object (опционально)
  * `httpReferrer` (String | [Referrer](structures/referrer.md)) (опционально) - HTTP Referrer.
  * `userAgent` String (optional) - A user agent originating the request.
  * `extraHeaders` String (optional) - Extra headers separated by "\n"
  * `postData` ([UploadRawData](structures/upload-raw-data.md) | [UploadFile](structures/upload-file.md))[] (optional)
  * `baseURLForDataURL` String (опционально) - Базовый URL (с разделителем пути), для файлов, которые будут загружены по URL данных. Это необходимо, только если указанный `url` это URL данных и необходимо загрузить другие файлы.

Возвращает `Promise<void>` - промис будет разрешен, когда страница завершит загрузку (см. [`did-finish-load`](web-contents.md#event-did-finish-load)), и отклоняет, если страница не удачно загрузилась (см. [`did-fail-load`](web-contents.md#event-did-fail-load)).

Тоже, что и [`webContents.loadURL(url[, options])`](web-contents.md#contentsloadurlurl-options).

The `url` can be a remote address (e.g. `http://`) or a path to a local HTML file using the `file://` protocol.

To ensure that file URLs are properly formatted, it is recommended to use Node's [`url.format`](https://nodejs.org/api/url.html#url_url_format_urlobject) method:

```javascript
const url = require('url').format({
  protocol: 'file',
  slashes: true,
  pathname: require('path').join(__dirname, 'index.html')
})

win.loadURL(url)
```

You can load a URL using a `POST` request with URL-encoded data by doing the following:

```javascript
win.loadURL('http://localhost:8000/post', {
  postData: [{
    type: 'rawData',
    bytes: Buffer.from('hello=world')
  }],
  extraHeaders: 'Content-Type: application/x-www-form-urlencoded'
})
```

#### `win.loadFile(filePath[, options])`

* `filePath` String
* `options` Object (опционально)
  * `query` Record<String, String> (опционально) - переданная в `url.format()`.
  * `search` String (optional) - Passed to `url.format()`.
  * `hash` String (optional) - Passed to `url.format()`.

Возвращает `Promise<void>` - промис будет разрешен, когда страница завершит загрузку (см. [`did-finish-load`](web-contents.md#event-did-finish-load)), и отклоняет, если страница не удачно загрузилась (см. [`did-fail-load`](web-contents.md#event-did-fail-load)).

Same as `webContents.loadFile`, `filePath` should be a path to an HTML file relative to the root of your application.  See the `webContents` docs for more information.

#### `win.reload()`

Same as `webContents.reload`.

#### `win.setMenu(menu)` _Linux_ _Windows_

* `menu` Menu | null

Устанавливает `menu` в меню окна.

#### `win.removeMenu()` _Linux_ _Windows_

Удаляет меню окна.

#### `win.setProgressBar(progress[, options])`

* `progress` Double
* `options` Object (опционально)
  * `mode` String _Windows_ - Mode for the progress bar. Can be `none`, `normal`, `indeterminate`, `error` or `paused`.

Устанавливает значение прогресса на шкале прогресса. Valid range is [0, 1.0].

Удаляет индикатор прогресса, когда прогресс меньше 0; Изменяет в режим indeterminate, когда прогресс больше 1.

На платформе Linux поддерживается только рабочая среда Unity, Вам необходимо указать имя файла `*.desktop` в поле `desktopName` в `package.json`. По умолчанию будет предполагаться `{app.name}.desktop`.

На Windows режим может быть передан. Принимаемые значения: `none`, `normal`, `indeterminate`, `error` и `paused`. Если Вы вызовете `setProgressBar` без установленного режима (но со значением в пределах допустимого диапозона), будет предполагаться `normal`.

#### `win.setOverlayIcon(overlay, description)` _Windows_

* `overlay` [NativeImage](native-image.md) | null - иконка, которая будет отображаться в правом краю иконки на панели задач. Если параметр `null`, оверлей будет очищен
* `description` String - описание, которое будет представлено для доступности чтения с экрана

Устанавливает 16x16 пиксельный оверлей поверх текущей иконки в панели задач, обычно используется для передачи какого-либо статуса приложения или пассивного уведомления пользователя.

#### `win.setHasShadow(hasShadow)`

* `hasShadow` Boolean

Устанавливает, будет ли окно иметь тень.

#### `win.hasShadow()`

Возвращает `Boolean` - есть ли у окна тень.

#### `win.setOpacity(opacity)` _Windows_ _macOS_

* `opacity` Number - между 0.0 (полная прозрачность) и 1.0 (полная видимость)

Sets the opacity of the window. On Linux, does nothing. Out of bound number values are clamped to the [0, 1] range.

#### `win.getOpacity()`

Возвращает `Number` - между 0.0 (полная прозрачность) и 1.0 (полная видимость). On Linux, always returns 1.

#### `win.setShape(rects)` _Windows_ _Linux_ _Экспериментально_

* `rects` [Rectangle[]](structures/rectangle.md) - Sets a shape on the window. Passing an empty list reverts the window to being rectangular.

Установка формы окна, которая определяет область в окне, где система разрешает отрисовку и взаимодействие пользователя. Вне данного региона ни один пиксель не отрисуется и ни одно событие мыши не будет зарегистрировано. Вне региона события мыши не будут получены этим окном, но будет передаваться чему-либо позади окна.

#### `win.setThumbarButtons(buttons)` _Windows_

* `buttons` [ThumbarButton[]](structures/thumbar-button.md)

Возвращает `Boolean` - успешно ли добавлены кнопки

Добавляет панель миниатюр, с определенным набором кнопок, на слой кнопок в изображении эскиза окна в панели задач. Возвращает объект `Boolean`, который указывает успешно ли добавлена панель миниатюр.

Количество кнопок в панели миниатюр не должно быть больше, чем 7, из-за ограничений. После установки панели миниатюр, панель не может быть удалена, из-за ограничений платформы. Но Вы можете вызвать метод с пустым массивом, чтобы убрать кнопки.

`buttons` является массивом объектов `Button`:

* `Button` Object
  * `icon` [NativeImage](native-image.md) - иконка, отображаемая на панели миниатюр.
  * `click` Function
  * `tooltip` String (опционально) - текст всплывающей подсказки на кнопке.
  * `flags` String[] (опционально) - Управление конкретными состояниями и поведением кнопки. По умолчанию, является `['enabled']`.

`flags` — это массив, который может включать следующие `строки`:

* `enabled` - кнопка активна и доступна пользователю.
* `disabled` - Кнопка отключена. Она присутствует, но имеет неактивное визуальное состояние и не будет реагировать на действия пользователя.
* `dismissonclick` - когда кнопка нажата, окно панели миниатюр закрывается немедленно.
* `nobackground` - не рисует границы кнопок, использует только изображение.
* `hidden` - кнопка не отображается пользователю.
* `noninteractive` - Кнопка включена, но не интерактивна; ни одно состояние кнопки не отображается. Это значение предназначено для случаев, когда кнопка используется в уведомлении.

#### `win.setThumbnailClip(region)` _Windows_

* `region` [Rectangle](structures/rectangle.md) - область окна

Устанавливает область окна, которая будет показана в панели миниатюр, при наведении мыши на окно в панели задач. Вы можете сбросить панель миниатюры, чтобы показывалось окно полностью, указав пустую область: `{ x: 0, y: 0, width: 0, height: 0 }`.

#### `win.setThumbnailToolTip(toolTip)` _Windows_

* `toolTip` String

Устанавливает всплывающую подсказку, которая будет отображена, при наведении мыши на панель миниатюры окна в панели задач.

#### `win.setAppDetails(options)` _Windows_

* `options` Object
  * `appId` String (опционально) - [App User Model ID](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391569(v=vs.85).aspx) окна. Это должно быть установлено, иначе остальные параметры не будут иметь никакого эффекта.
  * `appIconPath` String (опционально) - [иконка перезапуска](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391573(v=vs.85).aspx) окна.
  * `appIconIndex` Integer (optional) - Index of the icon in `appIconPath`. Ignored when `appIconPath` is not set. По умолчанию `0`.
  * `relaunchCommand` String (опционально) - [команда перезапуска](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391571(v=vs.85).aspx) окна.
  * `relaunchDisplayName` String (опционально) - [отображаемое имя перезапуска](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391572(v=vs.85).aspx) окна.

Устанавливает свойства для кнопки окна на панели задач.

**Note:** `relaunchCommand` and `relaunchDisplayName` must always be set together. If one of those properties is not set, then neither will be used.

#### `win.showDefinitionForSelection()` _macOS_

Тоже, что и `webContents.showDefinitionForSelection()`.

#### `win.setIcon(icon)` _Windows_ _Linux_

* `icon` [NativeImage](native-image.md) | String

Меняет иконку окна.

#### `win.setWindowButtonVisibility(visible)` _macOS_

* `visible` Boolean

Устанавливает, должны ли быть видны кнопки контроля окна.

#### `win.setAutoHideMenuBar(hide)`

* `hide` Boolean

Sets whether the window menu bar should hide itself automatically. Once set the menu bar will only show when users press the single `Alt` key.

If the menu bar is already visible, calling `setAutoHideMenuBar(true)` won't hide it immediately.

#### `win.isMenuBarAutoHide()`

Returns `Boolean` - Whether menu bar automatically hides itself.

#### `win.setMenuBarVisibility(visible)` _Windows_ _Linux_

* `visible` Boolean

Sets whether the menu bar should be visible. If the menu bar is auto-hide, users can still bring up the menu bar by pressing the single `Alt` key.

#### `win.isMenuBarVisible()`

Returns `Boolean` - Whether the menu bar is visible.

#### `win.setVisibleOnAllWorkspaces(visible[, options])`

* `visible` Boolean
* `options` Object (опционально)
  * `visibleOnFullScreen` Boolean (optional) _macOS_ - Sets whether the window should be visible above fullscreen windows.
  * `skipTransformProcessType` Boolean (optional) _macOS_ - Calling setVisibleOnAllWorkspaces will by default transform the process type between UIElementApplication and ForegroundApplication to ensure the correct behavior. However, this will hide the window and dock for a short time every time it is called. If your window is already of type UIElementApplication, you can bypass this transformation by passing true to skipTransformProcessType.

Устанавливает видимость окна на всех рабочих местах.

**Примечание:** Этот API ничего не делает в Windows.

#### `win.isVisibleOnAllWorkspaces()`

Возвращает `Boolean` - видно ли окно на всех рабочих местах.

**Примечание:** Данный API всегда возвращает false в Windows.

#### `win.setIgnoreMouseEvents(ignore[, options])`

* `ignore` Логическое значение
* `options` Object (опционально)
  * `forward` Boolean (optional) _macOS_ _Windows_ - If true, forwards mouse move messages to Chromium, enabling mouse related events such as `mouseleave`. Only used when `ignore` is true. If `ignore` is false, forwarding is always disabled regardless of this value.

Включает для окна игнорирование событий от мыши.

Все события мыши, произошедшие в этом окне, будут переданы окну позади этого окна, но, если это окно сфокусировано, оно все еще будет получать события клавиатуры.

#### `win.setContentProtection(enable)` _macOS_ _Windows_

* `enable` Boolean

Предотвращает захват содержимого окна другими приложениями.

On macOS it sets the NSWindow's sharingType to NSWindowSharingNone. On Windows it calls SetWindowDisplayAffinity with `WDA_EXCLUDEFROMCAPTURE`. For Windows 10 version 2004 and up the window will be removed from capture entirely, older Windows versions behave as if `WDA_MONITOR` is applied capturing a black window.

#### `win.setFocusable(focusable)` _macOS_ _Windows_

* `focusable` Boolean

Меняет, может ли окно быть сфокусировано.

На macOS он не снимает фокус с окна.

#### `win.isFocusable()` _macOS_ _Windows_

Returns whether the window can be focused.

#### `win.setParentWindow(parent)`

* `parent` BrowserWindow | null

Устанавливает `parent` как родителя текущего окна, передав `null` превратит текущее окно в окно верхнего уровня.

#### `win.getParentWindow()`

Возвращает `BrowserWindow` - родительское окно.

#### `win.getChildWindows()`

Возвращает `BrowserWindow[]` - все дочерние окна.

#### `win.setAutoHideCursor(autoHide)` _macOS_

* `autoHide` Boolean

Контролирует скрытие курсора при вводе текста.

#### `win.selectPreviousTab()` _macOS_

Выбирает предыдущую вкладку, когда нативные вкладки включены и в окне присутствуют другие вкладки.

#### `win.selectNextTab()` _macOS_

Выбирает следующую вкладку, когда нативные вкладки включены и в окне присутствуют другие вкладки.

#### `win.mergeAllWindows()` _macOS_

Объединяет все окна в одно окно с множественными вкладками, когда нативные вкладки включены и в присутствуют открытые окна больше, чем 1.

#### `win.moveTabToNewWindow()` _macOS_

Перемещает текущую вкладку в новое окно, если нативные вкладки включены и присутствует больше, чем одна вкладка, в текущем окне.

#### `win.toggleTabBar()` _macOS_

Переключает видимость вкладки, если включены нативные вкладки и присутствует только одна вкладка в текущем окне.

#### `win.addTabbedWindow(browserWindow)` _macOS_

* `browserWindow` BrowserWindow

Добавляет окно, как вкладку, в это окно, после вкладки экземпляра окна.

#### `win.setVibrancy(type)` _macOS_

* `type` String | null - Может быть `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light`, `ultra-dark`, `header`, `sheet`, `window`, `hud`, `fullscreen-ui`, `tooltip`, `content`, `under-window` и `under-page`. See the [macOS documentation][vibrancy-docs] for more details.

Adds a vibrancy effect to the browser window. Passing `null` or an empty string will remove the vibrancy effect on the window.

Обратите внимание, что `appearance-based`, `light`, `dark`, `medium-light` и `ultra-dark` устарела и будет удалена в следующей версии macOS.

#### `win.setTrafficLightPosition(position)` _macOS_

* `position` [Point](structures/point.md)

Set a custom position for the traffic light buttons in frameless window.

#### `win.getTrafficLightPosition()` _macOS_

Returns `Point` - The custom position for the traffic light buttons in frameless window.

#### `win.setTouchBar(touchBar)` _macOS_

* `touchBar` TouchBar | null

Sets the touchBar layout for the current window. Specifying `null` or `undefined` clears the touch bar. This method only has an effect if the machine has a touch bar and is running on macOS 10.12.1+.

**Примечание:** TouchBar API в настоящее время является экспериментальным и может быть изменен или удален в будущих версиях Electron.

#### `win.setBrowserView(browserView)` _Экспериментально_

* `browserView` [BrowserView](browser-view.md) | null - Attach `browserView` to `win`. If there are other `BrowserView`s attached, they will be removed from this window.

#### `win.getBrowserView()` _Экспериментально_

Returns `BrowserView | null` - The `BrowserView` attached to `win`. Returns `null` if one is not attached. Throws an error if multiple `BrowserView`s are attached.

#### `win.addBrowserView(browserView)` _Экспериментально_

* `browserView` [BrowserView](browser-view.md)

Заменяет метод setBrowserView, для поддержки работы с множественными видами браузера.

#### `win.removeBrowserView(browserView)` _Экспериментально_

* `browserView` [BrowserView](browser-view.md)

#### `win.setTopBrowserView(browserView)` _Экспериментально_

* `browserView` [BrowserView](browser-view.md)

Raises `browserView` above other `BrowserView`s attached to `win`. Throws an error if `browserView` is not attached to `win`.

#### `win.getBrowserViews()` _Экспериментально_

Возвращает `BrowserView[]` - массив всех BrowserViews которые были присоединены с `addBrowserView` или `setBrowserView`.

**Примечание:** BrowserView API в настоящее время экспериментально и может измениться или быть удалено в будущих релизах Electron.

[runtime-enabled-features]: https://cs.chromium.org/chromium/src/third_party/blink/renderer/platform/runtime_enabled_features.json5?l=70
[page-visibility-api]: https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API
[quick-look]: https://en.wikipedia.org/wiki/Quick_Look
[vibrancy-docs]: https://developer.apple.com/documentation/appkit/nsvisualeffectview?preferredLanguage=objc
[window-levels]: https://developer.apple.com/documentation/appkit/nswindow/level
[chrome-content-scripts]: https://developer.chrome.com/extensions/content_scripts#execution-environment
[event-emitter]: https://nodejs.org/api/events.html#events_class_eventemitter
[overlay-javascript-apis]: https://github.com/WICG/window-controls-overlay/blob/main/explainer.md#javascript-apis
[overlay-css-env-vars]: https://github.com/WICG/window-controls-overlay/blob/main/explainer.md#css-environment-variables
