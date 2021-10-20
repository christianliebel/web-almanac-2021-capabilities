# Capabilities
Capabilities are new web platform APIs that unlock entirely new use cases for web applications. Those new APIs are especially important for [Progressive Web Apps (PWA)](../pwa), a web-based application model. A PWA is a web app that can be installed to the system, even runs offline and launches very fast. To integrate with the underlying operating system, PWAs can only use web platform APIs. While some native features have already been exposed to the web (e.g., geolocation, gamepad or webcam access), many APIs were still missing (e.g., file system or clipboard access).

The [Capabilities Project](https://www.chromium.org/teams/web-capabilities-fugu) (codename Fugu) is a joint effort by the Microsoft, Intel, Google and other Chromium contributors. It tries to bridge the gap between native applications and web apps by designing and implementing new powerful web platform APIs in a secure and privacy-preserving manner. As capabilities unlock more and more use cases, they lay the path for entire new application categories to finally make the shift to the web (e.g., IDEs, image editors or office applications). Over the last two years, the focus for the Fugu team was on capabilities for desktop productivity applications and hardware-related APIs. This chapter will discuss several of those new capabilities and analyze on how many different desktop and mobile websites they are used. As capabilities are rather interesting for app-like websites, their relative usage is comparatively low. This is why we are using absolute numbers in this chapter.

For security reasons, some APIs require a user gesture (i.e., a click or keypress) in order to function. As the HTTP Archive crawler does not support detecting such APIs, the source code of the websites within the HTTP Archive is analysed instead. For instance, the regular expression `/navigator\.share\s*\(/g` is matched against the website’s source to determine if it makes use of the Web Share API. While this method is not 100% accurate, it should provide sufficiently good results. The exact regexes for the 30 supported APIs can be found [in this source file](https://github.com/HTTPArchive/legacy.httparchive.org/blob/master/custom_metrics/fugu-apis.js). All data in this chapter is based on the July 2021 crawl.

Please note that most of the APIs presented here are so-called incubations. Unless noted, they are not (yet) W3C Recommendations, i.e., official web standards. Instead, these APIs are discussed within the Web Platform Incubator Community Group (WICG), where browser vendors and developers can discuss new features. Some APIs have already shipped in different browsers, others are only available on Chromium-based browsers, some behind a flag developers need to activate.

## File System Access API
The first productivity-related API is the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API). It allows your application to securely access the user's file system: The user can select one or more files they want to give the web app access to via a file picker. In particular, the API does not grant arbitrary access to the file system. When the user has selected a file, the application can either modify its contents or create it in case it didn’t exist.

### Write Access

The following code prompts the user to pick a file from the local file system. If successful, it writes the text `hello world` to it and closes the file.

```js
const handle = await window.showSaveFilePicker();
const writable = await handle.createWritable();
await writable.write('hello world');
await writable.close();
```

### Read Access

The following code prompts the user to pick a file from the file system. This time, as the user could choose to open more than one file, the result will be an array of files. By using the destructuring expression `[handle]`, you will receive the handle of the first selected file as the first element in the array.

```js
const [handle] = await window.showOpenFilePicker();
const blob = await handle.getFile();
const text = await blob.text();
console.log(text);
```

### Opening Directories

Also, the API allows web apps (e.g., integrated development environments) to access entire directories and create, update or delete existing files or folders.

```js
const handle = window.showDirectoryPicker();
```

At the time of this writing, the File System Access API is only available on Chromium-based browsers on desktop systems. Fortunately, the web platform offers fallback approaches to achieve a similar operation on mobile devices and other browsers as well: With the `input[type=file]` control, it’s possible to give a web application access to the binary contents of a file. The `a[download]` mechanism allows the user to download a dynamically generated file to the Downloads folder. Developers can also use Thomas Steiner's library [browser-fs-access](https://github.com/GoogleChromeLabs/browser-fs-access) that uses File System Access API if present and otherwise falls back to the alternative implementation.

Out of all 6,286,373 desktop and 7,491,840 mobile websites in the HTTP Archive, the File System Access API is used on 29 desktop and 23 mobile sites. Examples for those sites are the image editor [Excalidraw](https://excalidraw.com/) that allows you to sketch hand-drawn like diagrams and save them to the disk. Another example is [CorelDRAW.app](h), a web version of the image editing software CorelDRAW.

## Async Clipboard API
The [Async Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API) allows you to read and write data from or to the clipboard. Due to its asynchronous nature, it enables use cases like scaling down an image while pasting it—all without blocking the UI. It replaces less capable APIs like `document.execCommand()` that were previously used to interact with the clipboard.

### Write Access
The Async Clipboard API offers two methods to copy data to the clipboard: The shorthand method `writeText()` takes arbitrary plain text as an argument which is then copied to the clipboard. The `write()` method takes an array of clipboard items that could contain arbitrary data. WebKit currently restricts the supported data formats to plain text, HTML, URL lists, and PNG images.

```js
await navigator.clipboard.writeText('hello world');
await navigator.clipboard.write([new ClipboardItem('hello world')]);
```

### Read Access
Similar to copying data to the clipboard, there are two methods to paste data back from the clipboard: First another shorthand method called `readText()` that returns plain text from the clipboard. By using the `read()` method, you get access to all copied clipboard items in a supported data format.

```js
const item = await navigator.clipboard.readText();
const items = await navigator.clipboard.read();
```

For privacy reasons, the browser may show a permission prompt or another UI before granting the website access to the contents from the clipboard. The Async Clipboard API is available in Chrome, Edge, and Safari. Firefox currently only supports the `writeText()` method.

With 560,359 (8.91%) desktop and 618,062 (8.25%) mobile sites, the Async Clipboard API (`writeText()` method) is one of the most used Fugu APIs. The `write()` method is used on 1,180 desktop and 1,227 mobile sites. The commercial website [Clipping Magic](https://clippingmagic.com/) allows you to remove the background of an image with the help of an AI algorithm. You can simply paste the image to be cropped from the clipboard—with the help of the Async Clipboard API.

## Web Share API
The [Web Share API](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share) allows you to share text, a URL, or files from a web site or web application with other applications on the same system, e.g., mail clients or messengers. To do so, you can simply call the `navigator.share()` method. It takes an object with the data that should be shared with another application. The browser then opens the native share sheet where the user can select the target application from. The methods returns a promise that resolves in case the contents could be successfully shared, otherwise, it will be rejected.

```js
await navigator.share({
  files: picturesArray,
  title: 'Holiday pictures',
  text: 'Our holiday in the French Alps'
})
``` 

The Web Share API is supported by Safari on iOS and macOS, and Chrome and Edge on Windows and Chrome. It’s currently a [Working Draft](https://www.w3.org/TR/web-share/), the first stage for specifications on the W3C Recommendation track, at the Web Applications Working Group.

With 566,049 (9.00%) desktop and 642,507 (8.58%) mobile sites, the Web Share API is the most used Fugu API. For example, the [beta version of the PaintZ app](https://beta.paintz.app/) allows you to share a drawing with another locally installed application via the save dialog.

## URL Handlers and Declarative Link Capturing
The last two productivity-related capabilities described in this chapter are URL Handlers and Declarative Link Capturing, additional methods for an even deeper integration with the operating system.

### URL Handling
With the help of [URL Handling](https://web.dev/pwa-url-handler/), PWAs can register themselves as handlers for certain URL schemes upon installation, e.g. for `https://*.example.com`. When the user opens a URL that matches this scheme, the installed PWA will open instead of a new browser tab. URL Handling is an extension of the [Web Application Manifest](https://www.w3.org/TR/appmanifest/), a file that contains metadata for web applications. To register for URL schemes, you have to add the `url_handlers` property to your manifest. This property takes an array containing objects with an `origin` property.

```json
{
  "url_handlers": [{
    "origin": "https://*.example.com"
  }]
}
```

If you want to register for origins other than your web app’s origin, you need to [verify your ownership of them](https://web.dev/pwa-url-handler/#the-web-app-origin-association-file). The API is at a rather early stage: It’s only supported on Chromium-based browsers on the desktop. URL Handling is currently available as an Origin Trial. This means that the capability is not generally available yet. Instead, developers need to register for an Origin Trial token first. More information can be found in the [Origin Trials Guide for Web Developers](https://github.com/GoogleChrome/OriginTrials/blob/gh-pages/developer-guide.md).

44 desktop and 41 mobile websites make use of URL Handling. For example, the Pinterest PWA registers itself as a URL handler for the different Pinterest origins (e.g., `*.pinterest.com` and `*.pinterest.de`) on installation.

### Declarative Link Capturing
With the help of [Declarative Link Capturing](https://web.dev/declarative-link-capturing/), you can further control how PWAs should behave when they are opened. For instance, an office application may want to open another window when a new document is opened, while a music player wants to keep its single window. Therefore, Declarative Link Capturing defines three different modes:

1. `none` does not capture the link at all (the default)
2. `new-client` opens a new window for the PWA
3. `existing-client-navigate`  navigates an existing client to the new URL or opens a new window if no client exists

Declarative Link Capturing is also an extension of the Web Application Manifest. To use it, you need to add the `capture_links` property to your manifest. This property takes a string or an array of strings matching the three modes from above. If you use an array, the browser will fallback to the next entry if it doesn’t support a mode.

```json
{
  "capture_links": [
    "existing-client-navigate",
    "new-client",
    "none"
  ]
}
```

This capability is also at an early stage. It is only supported on Chromium-based browsers on Chrome OS. Currently, 36 desktop sites and 11 mobile sites make use of this capability, for example [Periodex](https://periodex.co/), a PWA showing the periodic table of elements. This app uses the `capture_links` configuration as shown above: If supported, the browser should reuse the existing window, otherwise open a new one, and if that’s not supported, it should behave as normal.

## Hardware APIs
The next set of capabilities focuses on hardware-related APIs. In Chromium-based browsers, there are many APIs to access hardware interfaces, including but not limited to Web USB, Web Bluetooth and Web Serial API. Furthermore, the Generic Sensor API allows you to read out device sensors. All capabilities discussed in this section are only available on Chromium-based browsers and on systems where the according interface or sensor is present.

### Web USB API
The [Web USB API](https://web.dev/usb/) allows developers to access USB devices without any drivers or third-party applications. This capability is interesting for firmware updaters that would have to be implemented as native apps. To access USB devices, you need to call the `navigator.usb.requestDevice()` method. It takes an object defining filters for the list of all connected USB devices. You need to at least specify the `vendorId`. The user is presented with a device picker where they can choose a matching device from. From there, you can begin a device session.

```js
try {
  const device = await navigator.usb.requestDevice({ filters: [{ vendorId: 0x8086 }] });
  console.log(device.productName);
  console.log(device.manufacturerName);	
} catch (err) {
  console.log(err);
}
```

The API is generally available on Chromium-based browsers since version 61 from September 2017. 182 desktop and 155 mobile sites use this API, for example, the PWA [Vysor](https://app.vysor.io/#/) that allows you to mirror the screen of an Android or iOS device—all without installing any additional software on your computer.

### Web Bluetooth API
The [Web Bluetooth API](https://web.dev/bluetooth/) allows you to communicate with nearby Bluetooth Low Energy devices using the Generic Attribute Profile (GATT). To find a matching device, call the `navigator.bluetooth.requestDevice()` method. In the following example, the list of Bluetooth devices is filtered by whether they offer a battery service or not. The user is presented with a device picker they have to pick a device from. Afterward, you can connect to the remote device and gather the data.

```js
try {
  const device = await navigator.bluetooth.requestDevice({ filters: [{ services: ['battery_service'] }] });
  console.log(device.name);
} catch (err) {
  console.log(err);
}
```

The API is generally available in Chromium-based browsers on Chrome OS, Android, macOS, and Windows. On Linux, the API is available behind a flag. 71 desktop and 45 mobile sites make use of this capability. For instance, the [Brewfather](https://web.brewfather.app/) app targeted at home-brewers allows them to send a beer recipe over to a Bluetooth-enabled brewing system. Again, all without installing any third-party software.

### Web Serial API
The [Web Serial API](https://web.dev/serial/) allows you to connect with serial devices such as microcontrollers. To do so, you can simply call the `navigator.serial.requestPort()` method. You can optionally pass in a method to filter the device list. The user is presented with a device picker they can choose a device from. Next, you can open the connection by calling the `open()` method on the port.

```js
try {
  const port = await navigator.serial.requestPort();
  await port.open({ baudRate: 9600 });
} catch (err) {
  console.log(err);
}
```

This capability is relatively new, as it shipped with Chromium 89 in March 2021. Currently, 15 desktop and 14 mobile sites make use of Web Serial API, including the [Duino App](https://duino.app/) that allows you to develop programs for Arduino and ESP microcontrollers. They are compiled on a remote server and then uploaded to a connected board via the Web Serial API.

### Generic Sensor API
Finally, the [Generic Sensor API](https://web.dev/generic-sensor/) allows you to read sensor data from the device’s sensors, such as the accelerometer, gyroscope, or orientation sensor. To access a sensor, create a new instance of a sensor class, e.g., `Accelerometer`. The constructor takes a configuration object with the requested frequency. By attaching to the `onreading` and `onerror` callbacks, you can get notified for updated sensor values, or errors respectively. Finally, you need to start the reading by calling the `start()` method.

```js
try {
  const accelerometer = new Accelerometer({ frequency: 10 });
  accelerometer.onerror = (event) => {
    console.log(event);
  };
  accelerometer.onreading = (e) => {
    console.log(e);
  };
  accelerometer.start();
} catch (err) {
  console.log(err);
}
```

The capability is supported by Chromium browsers starting from version 67. The relative orientation sensor is used on 824 desktop and 831 mobile sites, the linear acceleration sensor on 257 desktop and 237 mobile sites, and the gyroscope is used on 36 desktop and 22 mobile sites. An example application is [VDO.Ninja](https://obs.ninja/), the former OBS Ninja. This software allows you to remotely connect with video broadcasting software such as OBS. The app allows you the connected software to read sensor data from the device.  For example, to capture a smartphone’s movement when streaming VR content.

## Sites using most capabilities
In the analysis, also the sites using the most capabilities from the HTTP Archive set were identified. The crawler is capable of identifying 30 Fugu APIs.

1. The number one site is [whatwebcando.today](https://whatwebcando.today/), which uses 28 APIs. It showcases different HTML5 device integration APIs by providing a live demo for every capability. Naturally, the number of used APIs is very high. In the result set, there’s a similar site called [whatpwacando.today](https://whatpwacando.today/)  which showcases PWA capabilities and uses eight APIs.
2. The [PolisNotis](https://polisnotis.se/) PWA shows police notices in Sweden. It uses ten APIs, including the Declarative Link Capturing API to define that the PWA should always open a new window when clicking a PWA-related link. The Web Share API is used in the source code, but the sharing functionality does not seem to be exposed to the UI.
3. The website [System Scanner](https://system-scanner.net/) uses nine APIs: It shows an overview of the system information exposed by the browser or the user’s IP address, including sensor information provided by the Generic Sensor API.
4. Eight sites use eight Fugu APIs: One of them is the aforementioned [Excalidraw](https://excalidraw.com/), an online drawing tool for creating drawings in a hand-drawn style. This is a traditional productivity app which benefits from the new capabilities.

The results also include sites that aren’t pro-actively using the APIs. For example, some sites ship library code that could theoretically access the capabilities. Some sites even check for the Fugu APIs to determine the user’s browser.

## Conclusion
Capabilities help move the web forward by unlocking more and more use cases for developers. As this chapter shows, developers use the new web platform APIs to build powerful applications. In contrast to their native counter parts, those applications don’t need to be installed to the system and don’t require any additional third-party runtimes or plug-ins to work. They run on any platform that can run a reasonably powerful browser. This makes writing applications much easier and more efficient for developers.

#article/almanac/2021
