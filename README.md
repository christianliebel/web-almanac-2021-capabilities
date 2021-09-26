# Capabilities

This chapter discusses new web platform APIs that unlock entirely new use cases on the web. These APIs are also referred to as capabilities. Last year's chapter presented several new web platform APIs together with their usage statistics primarily based on telemetry data collected by Blink. This year's chapter discusses the most important capabilities for productivity applications. These kind of applications tightly integrate with a file-based workflow: Users double-click a file, edit it and save it back to the file system. Apps like Visual Studio Code, Adobe Photoshop or Microsoft Excel heavily rely on this workflow. By accessing the clipboard, those apps can exchange data with other applications on the system.

Capabilities are especially important for Progressive Web Apps (PWA). This is a web-based application model that allows you to write web apps that can be installed to the system and run offline. As capabilities unlock more and more use cases, they lay the path for entire new application categories to finally make the shift to the web. Most of the new web platform APIs are created within Project Fugu, a joint effort of Microsoft, Intel and Google to brigde the gap between native applications and web apps. Over the two years, the focus for the Fugu teams was on desktop capabilities. The Fugu team also makes sure that the security and privacy of the user is maintained despite of the new powerful APIs.

After discussing some of the new productivity and hardware-related APIs, we will compare how the usage statistics have changed for the Async Clipboard and Storage Manager API that were presented last year. As most of the capabilities cannot be detected by the HTTPArchive crawler, this year's mechanism is to analyze the source code and finding usages of the APIs.

## File System Access API

The first API is the File System Access API. It allows your application to access the user's file system in a secure manner: The user has to select one or more files they want to give the web app access to. The API does not grant arbitrary access to the file system. Next, the application can modify the contents of a previously opened file.

The following code prompts the user to pick a file from the local file system. If successful, it writes the text `hello word` to it and closes the file.

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

At the time of this writing, the File System Access API is only available on Chromium-based browsers and desktop operating systems. Fortunately, the web platform offers fallback approaches to achieve a similar operation on mobile devices and other browsers as well.

(TODO: Describe alternative)

Developers can also use Thomas Steiner's library browser-fs-access that uses File System Access API if present and otherwise falls back to the alternative implementation.

(TODO: Usage statistics)

## Handlers and Link Capturing

The next section describes additional methods to integrate with the operating system with the help of file, URI and protocol handlers as well as desktop link capturing.

### File Handling API

The File Handling API allows you to register your PWA for a certain file type or extension.

- Manifest extension
- launch queue

### URL Handlers

### Protocol Handlers

### Declarative Link Capturing

## Window Controls Overlay

Some applications want to extend their controls into the window chrome. The Window Controls Overlay capability allows applications to extend the window's titlebar. The security is maintained by disallowing developers to hide the window controls provided by the operating system.

To use this capability, the Fugu team extended the Web App Manifest by a new property called `display_overrides`. 

- Manifest extension (display overrides)
- CSS

## Hardware APIs

Many applications also access hardware, such as USB or bluetooth devices, NFC tags, serial devices or HID apps. Furthermore, 

### Web USB API

### Web Bluetooth API

### Web NFC API

### Web Serial API

### Web HID API

### Sensor APIs

## Local Font Access API

The Local Font Access API allows you to request the list of locally installed fonts. This capability is important for image editors or presentation programs.

```js
TODO: Code
```

The list of local fonts is problematic from a privacy point of view, as it may reveal more information about the user (e.g., corporate fonts) or even uniquely identify the user. This is prevented by allowing the user to select the fonts they want to share with the web app. They can also choose to share all installed fonts with the website.

## Async Clipboard API

The Async Clipboard API allows you to read and write data from or to the clipboard. Due to its asynchronous nature, it enables use cases like converting image data while pasting without blocking the UI.

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

## Storage Manager API

```js
TODO
```

- Comparison with last year

## Sites using most capabilities

These are the sites that use the most Fugu capabilities:

1. The site using the most APIs (28) on both desktop and mobile is whatwebcando.today, an overview website that demonstrates the ability of the modern web.
2. The Polis Notis app shows crimes in Sweden. (which?)
3. Eight sites use 8 Fugu APIs. One of them is Excalidraw, an online image editor.

## Conclusion

The support for …

