# Chat Replay

A private, offline viewer that turns an exported WhatsApp chat into a familiar chat screen you can scroll, search, and revisit. Everything runs in your browser. Your messages and photos are never uploaded anywhere.

> This is a personal project, built as a way to keep an important conversation in a form that is easy to read years later. It is not affiliated with or endorsed by WhatsApp or Meta.


## Features

- **Reads real exports.** Android and iPhone `.txt` files, and `.zip` exports that include media.
- **Handles messy formats.** 12-hour and 24-hour time, `DD/MM` and `MM/DD` dates (auto-detected, with a one-click switch), multi-line messages, system messages, deleted and edited messages.
- **Your media folder.** Pair a "Without media" `.txt` with the folder of photos, videos and voice notes, and each file is placed back in its message.
- **Fast on large chats.** A virtualized list keeps scrolling smooth with 100,000+ messages.
- **Search and navigation.** Full-text search with highlighting, a jump-to-date calendar, a floating date label while scrolling, and a jump-to-latest button.
- **WhatsApp-style formatting.** `*bold*`, `_italic_`, `~strike~`, `` `mono` ``, clickable links, and large emoji-only messages.
- **Dark mode, wallpaper toggle, and print to PDF.**
- **Private by design.** No server, no analytics, no external fonts or CDNs. Data stays on your device.

## Tech stack

| Layer | Technology |
| --- | --- |
| Language | TypeScript (about 90% of the code), CSS, HTML |
| UI | React 19 |
| Build tool | Vite |
| Styling | Tailwind CSS plus hand-written CSS for the chat look |
| Large lists | react-virtuoso |
| Zip files | fflate |
| Local storage | IndexedDB through `idb` |
| Testing | Vitest |

Roughly 1,300 lines of TypeScript, 140 lines of CSS, and a small HTML shell.

## How it works

```
 .txt / .zip  ──▶  Web Worker  ──▶  Parser  ──▶  Message[]  ──▶  Virtualized chat list
 + media folder         │                              │
                        │                              └──▶  Media matcher ──▶  Photo / video / audio
                        └── keeps the page responsive while a large file is read
```

1. **Import.** You drop a chat file (and optionally a media folder) on the page. A `.zip` is unpacked in the browser with `fflate`, and only the chat text is read at first. Media stays compressed until needed.
2. **Parse.** The text is parsed inside a Web Worker so the page never freezes. The parser joins multi-line messages, recognizes Android and iPhone line formats, removes invisible characters WhatsApp inserts, and converts every timestamp to a normalized ISO string. Timestamps are shown exactly as written in the file, with no timezone conversion.
3. **Date order.** If any date has a first number above 12, the file must be `DD/MM`. If any has a second number above 12, it must be `MM/DD`. If neither happens, the file is ambiguous, so the app shows a banner with a switch.
4. **Pick who you are.** The app asks which sender is you, so your messages appear on the right in green. The choice is remembered.
5. **Match media.** A "Without media" export only says `<Media omitted>`, with no file names. Android names its files like `IMG-20240312-WA0001.jpg`, so the matcher pairs files with omitted-media messages by date and order. It only does this on a day where the number of files equals the number of omitted messages, so a wrong photo never ends up in a bubble. Anything it cannot match safely stays as a "Media omitted" placeholder.
6. **Render.** Messages are flattened into a list with date chips and grouped bubbles, and shown with `react-virtuoso` so only what is on screen is drawn. Message text is turned into React elements, never inserted as HTML, so chat content cannot inject markup.

## Getting started

### Requirements

- Node.js 20 or newer (22 recommended)
- npm

### Run it on Windows with WSL (Ubuntu)

One-time setup:

```bash
sudo apt update && sudo apt install -y unzip curl git
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 22
```

Get the project and start it:

```bash
git clone https://github.com/YOUR_USERNAME/chat-replay.git
cd chat-replay
npm install
npm run dev
```

Open **http://localhost:5173** in your normal Windows browser. If the page does not load, run `npm run dev -- --host` instead.

Your browser runs on Windows, so you can choose the chat file and media folder directly from your Windows drives. You do not need to copy them into WSL.

### Run it on macOS or Linux

```bash
git clone https://github.com/YOUR_USERNAME/chat-replay.git
cd chat-replay
npm install
npm run dev
```

### Other commands

```bash
npm test          # run the test suite
npm run build     # type-check and create a production build in dist/
npm run preview   # preview the production build locally
```

## Using it

1. **Export your chat from WhatsApp.**
   - Android: open the chat, tap the three dots, More, Export chat, then choose Without media or Include media.
   - iPhone: open the chat, tap the contact name, Export Chat, then choose Without Media or Attach Media.
2. **Copy the file to your computer.** If you want photos and videos, also copy the media folder. On Android it is usually under `WhatsApp/Media/`.
3. **Open the app** and drop the chat file and media folder on the page, or use the two buttons.
4. **Choose which sender is you.** Then scroll, search, or jump to any date.

There is also a "Try the sample chat" button if you just want to see how it looks.

## Project structure

```
src/
  lib/
    parseChat.ts         Parses export text into messages
    detectDateFormat.ts  Decides DD/MM or MM/DD
    matchMedia.ts        Pairs omitted-media messages with files
    pickFiles.ts         Reads dropped folders, finds the chat file
    zip.ts               Reads chat and media from zips and folders
    formatText.tsx       WhatsApp text formatting without innerHTML
    dates.ts             Date and time helpers
  workers/
    parser.worker.ts     Runs the parser off the main thread
  store/
    chatStore.ts         Saves the last chat and settings in IndexedDB
  components/            Chat screen, bubbles, search, calendar, import screen
  styles/index.css       Theme colors, bubbles, wallpaper, dark mode
tests/                   Parser, formatter, and media-matching tests
```

## Testing

The test suite covers the parts most likely to break:

- the parser across Android and iPhone formats, 12h and 24h time, and messages that cross midnight
- date-order detection and the manual override
- multi-line messages, system messages, emoji, Hindi text, and malformed input that must not crash
- parsing speed on 100,000 messages
- text formatting, link safety, and HTML escaping
- media matching, including days where counts do not agree

```bash
npm test
```

## Privacy

- Nothing is sent to a server. The app has no backend.
- No analytics, trackers, external fonts or CDNs.
- The last chat and your settings are stored in your browser's IndexedDB so you can continue later. Menu, then "Clear all data", removes them.
- Media files are read from your disk only when a message scrolls into view, and are not copied.

## Known limitations

- Media matching by date and order only works for Android-style file names, and only on days where the counts agree.
- The media folder is not saved between visits, so pick it again after continuing a saved chat.
- A sender name that contains a colon followed by a space cannot be told apart from message text.
- Search counts messages, not every match inside one message.
- WhatsApp changes its export format from time to time. If a file does not parse, open the menu and check "Skipped lines".

## Roadmap

- [ ] Replay mode, with messages appearing one at a time
- [ ] Statistics panel (who wrote more, busiest day, top emojis)
- [ ] Custom avatars
- [ ] "On this day" reminders
- [ ] Remember the media folder between visits
