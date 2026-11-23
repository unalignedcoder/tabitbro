<picture>
  <source media="(prefers-color-scheme: dark)" srcset="icons/icon-white128.png">
  <img width="128" height="128" alt="TabItBro" src="icons/icon128.png">
</picture>

# TabItBro

Keep the page you are on, and open new destinations in a new tab.

<img width="289" height="336" alt="2026-11-20 18_11_08-Technocracy" src="https://github.com/user-attachments/assets/669bd70e-6275-4f5c-9522-f5570191c8f7" />


TabItBro is a small Chrome extension (Manifest V3) that makes these navigations open in a new tab instead of replacing the current page:

1. Links that point to a different domain (subdomains count as the same domain)
2. URLs typed or pasted into the address bar, plus address-bar searches
3. Bookmarks
4. Items clicked on the `chrome://history` page

Each one is a checkbox on the options page. New tabs can also be opened in the background. The options page follows the browser or system light/dark theme, and so does the toolbar icon (blue on light themes, white on dark ones). Clicking the toolbar icon opens the options.

# Download and Install

Download and install the latest version from The Chrome or [Edge](https://microsoftedge.microsoft.com/addons/detail/dafmiicjoamppjffiegmjdedbdfhepee) stores.

Otherwise from the [Releases](https://github.com/unalignedcoder/tabitbro/releases) page, then:

1. Open `chrome://extensions` (`edge://extensions`, `brave://extensions`…) and turn on **Developer mode**.
2. Click **Load unpacked** and select the extension folder.
3. To change settings, open the extension's **Details** page and click **Extension options**.

It works in Chrome, Edge, Brave, Vivaldi and other Chromium browsers.

## How it works

- **External links (#1):** a content script intercepts link clicks before the browser navigates and opens the link in a new tab. This is clean: the current page never changes.
- **Address bar, bookmarks and history (#2–#4):** these navigations happen in the browser itself, outside any web page. The service worker spots them as they are committed, opens the URL in a new tab and sends the original tab back to where it was.

## Toolbar icon

Chrome gives extensions no way to read the toolbar color, and the service worker can't check the color scheme either. So the pages report it instead: the content script and the options page read `prefers-color-scheme` and tell the service worker when it changes, and the worker switches the icon with `chrome.action.setIcon`. This follows Chrome's *Mode* setting (light / dark / device). A custom Chrome theme that is dark while the mode is light still gets the blue icon.

## Limitations

**Chrome has no API that lets an extension stop or redirect an address-bar, bookmark or history navigation before it happens.** The only thing an extension can do is react once the navigation has started. So for options #2–#4, TabItBro has to let the page **open in the same tab first, then go back**:

- The current tab briefly starts loading the new page, then goes back to the previous one, while the page opens in a new tab. You may see a short flicker.
- The site may receive one extra request (from the tab that went back).
- Because the tab really did navigate, the page you went back from may reload, and unsaved form input on it can be lost. Its back/forward history also gets the extra entry.
- When the current tab is an empty New Tab page there is nothing to go back to, so the navigation simply stays in that tab.

Other limitations:

- Content scripts don't run on `chrome://` pages or the Chrome Web Store, so #1 doesn't apply there.
- #1 only handles real `<a href>` clicks. It doesn't catch JavaScript navigations or form submissions.
- Ctrl-, Shift- and middle-clicks are left to the browser's normal handling.
- If address-bar navigations are sometimes missed, Chrome may have prerendered the page (*Settings → Performance → Preload pages*). Turn on **Debug logging** in the options and check the service worker console to see which transition types are reported.

## Permissions

TabItBro has a single purpose: open the kinds of navigation you choose in a new tab instead of the current one. It requests only what that needs:

| Permission | Why |
|---|---|
| `storage` | Saves the options, remembers the previous URL of each tab for the session, and remembers the light/dark theme for the toolbar icon. |
| `webNavigation` | Detects address-bar, bookmark and history navigations (`webNavigation.onCommitted` with its transition type), so they can be moved to a new tab. |
| `tabs` | Reads tab URLs to know which page a tab was on before a navigation, for example the `chrome://history` page or an empty New Tab page. Host permissions can't give access to `chrome://` URLs. |
| Content script on `http://*/*`, `https://*/*` | Checks link clicks on web pages, so links to a different domain open in a new tab before the page changes. |

There are no host permissions besides the content script, no remote code, no analytics, and no data leaves the browser. Options are kept in Chrome's synced storage. See the [privacy policy](privacy.md).

## Project layout

```
manifest.json    extension manifest (MV3)
background.js    service worker: address bar, bookmarks, history (#2–#4), opens tabs for #1
content.js       intercepts clicks on links to other domains (#1)
options.html/js  options page
icons/           extension icons (blue for light themes, white for dark ones)
```

## Changelog

### 1.0.3
- Renamed to TabItBro (the repository is now github.com/unalignedcoder/tabitbro)

### 1.0.2
- Options page: wider (560px) and no horizontal scrolling; text and images shrink to fit narrow dialogs
- Added a [privacy policy](privacy.md): the extension collects and sends no data

### 1.0.1
- Removed the unneeded `<all_urls>` host permission
- The content script runs only on http/https pages
- README lists every permission and why it is needed

### 1.0.0
- First release: opens links to other domains, address-bar URLs and searches, bookmarks and history items in a new tab (each can be turned on or off)
- Option to open new tabs in the background
- Options page and toolbar icon follow the light/dark theme
- Buy me a coffee button

## Support

TabItBro is free. If you find it useful, you can support its development:

<a href="https://buymeacoffee.com/unalignedcoder" target="_blank"><img src="icons/buymeacoffee.png" alt="Buy me a coffee" width="218" height="61"></a>

## License

[MIT](LICENSE)
