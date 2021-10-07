# Capabilities
This chapter discusses new web platform APIs, also referred to as capabilities, that unlock entirely new use cases on the web. Last year's chapter presented several of these new APIs together with their usage statistics primarily based on telemetry data collected by Blink. This year's chapter discusses the most important capabilities for productivity applications. These kind of applications tightly integrate with a file-based workflow: Users double-click a file, edit it and save it back to the file system. Apps like Visual Studio Code, Adobe Photoshop or Microsoft Excel heavily rely on this workflow. By accessing the clipboard, those apps can exchange data with other applications on the system.

TODO: PWA Link

Capabilities are especially important for Progressive Web Apps (PWA). This is a web-based application model that allows you to write web apps that can be installed to the system and run offline. As capabilities unlock more and more use cases, they lay the path for entire new application categories to finally make the shift to the web. Most of the new web platform APIs are created within Project Fugu, a joint effort of Microsoft, Intel and Google to brigde the gap between native applications and web apps. Over the two years, the focus for the Fugu teams was on desktop capabilities. The Fugu team also makes sure that the security and privacy of the user is maintained despite of the new powerful APIs.

After discussing some of the new productivity and hardware-related APIs, we will compare how the usage statistics have changed for the Async Clipboard and Storage Manager API that were presented last year. As most of the capabilities cannot be detected by the HTTPArchive crawler, this year's mechanism is to analyze the source code and finding usages of the APIs.

Please note that most of the APIs presented here are no W3C Recommendations, i.e., they aren’t official web standards yet. Instead, these APIs are discussed within the Web Platform Incubator Community Group (WICG), where browser vendors and developers can discuss new features. Some APIs have already shipped in different browsers, others are only available on Chromium-based browsers, some behind a flag.

## File System Access API
The first API is the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API). It allows your application to access the user's file system in a secure manner: The user can select one or more files they want to give the web app access to. The API does not grant arbitrary access to the file system. Next, the application can modify the contents of a previously opened file.

The following code prompts the user to pick a file from the local file system. If successful, it writes the text `hello world` to it and closes the file.

```js
const handle = await window.showSaveFilePicker();
const writable = await handle.createWritable();
await writable.write('hello world');
await writable.close();
```

(TODO: Screenshot)

The following code prompts the user to pick a file from the file system. This time, as the user could choose to open more than one file, the result will be an array of files. By using the destructuring expression `[handle]`, you will receive the handle of the first selected file as the first element in the array.

```js
const [handle] = await window.showOpenFilePicker();
const blob = await handle.getFile();
const text = await blob.text();
console.log(text);
```

Also, the API allows web apps (e.g., integrated development environments) to access entire directories and create, update or delete existing files or folders.

```js
const handle = window.showDirectoryPicker();
```

At the time of this writing, the File System Access API is only available on Chromium-based browsers and desktop operating systems. Fortunately, the web platform offers fallback approaches to achieve a similar operation on mobile devices and other browsers as well: With the `input[type=file]` control, it’s possible to give a web application access to the binary contents of a file. The `a[download]` mechanism allows the user to download a dynamically generated file to the Downloads folder. Developers can also use Thomas Steiner's library [browser-fs-access](https://github.com/GoogleChromeLabs/browser-fs-access) that uses File System Access API if present and otherwise falls back to the alternative implementation.

(TODO: Usage statistics)

## Handlers and Link Capturing
The next section describes additional methods to integrate with the operating system with the help of file, URI and protocol handlers as well as desktop link capturing. All APIs presented in this section are only available on Chromium-based browsers behind a flag.

### File Handling API

The [File Handling API](https://web.dev/file-handling/) allows you to register your PWA for a certain file type or extension. The registration takes place when installing the user installs the application on the system. The API is only available on desktop systems.

(Screenshot: Excalidraw)

The File Handling API consists of two parts: First of all, the Web App Manifest of the web application needs to be extended by the `file_handlers` property. This property takes an array of file handlers that consist of an `action` (i.e., the URL to call when the application is opened to handle a file), and an `accept` object. There are two different approaches on how to determine which type a file has. Windows traditionally uses file extensions, whereas Unix-based systems take a look at the magic number. As web-based applications should work regardless of the target platform, the File Handling API needs to support both approaches. This is where the `accept` object comes into play: It acts as a map with media types as a key and an array of file extensions assigned to them. In the following example, the application registers for files with the media type `text/plain` for Unix-based systems and `.txt` for Windows.

```json
{
  "name": "App",
  "file_handlers": [{
    "action": "/",
    "accept": {
      "text/plain": [".txt"]
    }
  }]
}
```

After installing, the application is now registered as a handler for PNG files. The second part is to get access to the files the application was opened with. Therefore, the API adds the `launchQueue` property to the global window object. By calling the `setConsumer()` method, the developer gets access to  the parameters the application was launched with. In the `files` property of the `params` object, the’s the array of file handles the application was opened with. By calling the `getFile()` method, one gets access to the binary data of this file—comparatively to the file open sample from above.

```js
window.launchQueue.setConsumer(async (params) => {
  const [handle] = params.files;
  if (handle) {
    const file = await handle.getFile();
    const text = await file.text();
    console.log(text);
  }
});
```

(Usage statistics)

(TODO: In order to use this API, you need to enable the flag `about:flags/#/file-handling`)

### URL Handlers

With the help of [URL Handling](https://web.dev/pwa-url-handler/), PWAs can also act as URL handlers…

(Usage statistics)

### Protocol Handlers

PWAs can also act as [Protocol handlers](https://web.dev/url-protocol-handler/)

(Usage statistics)

### Declarative Link Capturing

[Declarative Link Capturing](https://web.dev/declarative-link-capturing/)

(Usage statistics)

## Window Controls Overlay
Some applications want to extend their controls into the window chrome, e.g. (TODO). The [Window Controls Overlay](https://web.dev/window-controls-overlay/) capability allows applications to extend the window's titlebar. The security is maintained by disallowing developers to hide the window controls provided by the operating system.

To use this capability, the Fugu team extended the Web Application Manifest by a new property called `display_overrides`. 

- Manifest extension (display overrides)
- CSS

## Hardware APIs
Many applications also access hardware, such as USB or bluetooth devices, NFC tags, serial devices or HID apps. Furthermore, 

All APIs discussed in this section are only available on Chromium-based browsers and on systems where the according interface is present.

### Web USB API
The [Web USB API](https://web.dev/usb/) allows developers to access USB devices.

(Usage statistics)

### Web Bluetooth API
https://web.dev/bluetooth/

(Usage statistics)

### Web NFC API
https://web.dev/nfc/

Android only

(Usage statistics)

### Web Serial API
https://web.dev/serial/

(Usage statistics)

### Web HID API
https://web.dev/hid/

(Usage statistics)

### Generic Sensor API
https://web.dev/generic-sensor/

(Usage statistics)

## Local Font Access API
The [Local Font Access API](https://web.dev/local-fonts/) allows you to request the list of locally installed fonts. This capability is important for image editors or presentation programs.

```js
const status = await navigator.permissions.query({
    name: 'font-access',
});

if (status.state === 'denied')
    throw new Error('Cannot enumerate local fonts');

for await (const font of await navigator.fonts.query()) {
  console.log(font.family);
}
```

The list of local fonts is problematic from a privacy point of view, as it may reveal more information about the user (e.g., corporate fonts) or even uniquely identify the user. This is prevented by allowing the user to select the fonts they want to share with the web app. They can also choose to share all installed fonts with the website.

## Async Clipboard API
The [Async Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API) allows you to read and write data from or to the clipboard. Due to its asynchronous nature, it enables use cases like converting image data while pasting without blocking the UI.

With 8.91 % (desktop) and 8.25 % (mobile) sites, the Async Clipboard API is the second-most used Fugu API …

```js
await navigator.clipboard.write([new ClipboardItem('hello world')]);
await navigator.clipboard.writeText('hello world');
```

```js
const items = await navigator.clipboard.read();
const item = await navigator.clipboard.readText();
````

- Comparison with last year (new formats)
- TODO: Raw Clipboard Access

## Storage Manager API
```js
const { usage, quota } = await navigator.storage.estimate();
```

```js
const persisted = await navigator.storage.persist();
```

- Comparison with last year

[Storage Manager API](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager)

## Sites using most capabilities
These are the sites that use the most Fugu capabilities:

1. The site using the most APIs (28) on both desktop and mobile is whatwebcando.today, an overview website that demonstrates the ability of the modern web.
2. The Polis Notis app shows crimes in Sweden. (which?)
3. Eight sites use 8 Fugu APIs. One of them is Excalidraw, an online image editor.

## Conclusion
The support for …

#article/almanac/2021
