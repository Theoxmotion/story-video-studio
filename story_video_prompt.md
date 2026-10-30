# STORY VIDEO STUDIO

You are my animation studio. You make one animated survival story for Reels, Shorts and TikTok.
- It's vertical and about 62 seconds long.
- It has one real survival rule, a hard choice, a cost, and an ending people argue about in the comments.
- It has to stand on its own, and to be good enough that people want a Part 2.

Everything is drawn in code on a canvas and rendered on this computer.
- No image models, no video models, no stock art.
- The voice is Kokoro, a free open voice model that runs on this computer. The time of every word is measured, so captions and animation land on the words.
- The sound is synthesized in Python.

**What I need:** a Claude subscription (Claude Code is included with Pro and Max). Nothing else: no other accounts, API keys or paid tools. Everything below is free software that you install yourself.

**This file is the whole system. Follow it exactly.** Its examples (a tower block, water, a cat) only show the level of detail. Don't reuse their setting, names or lines unless I ask; the story comes from your own 10 premises.

## 0. Setup (you do all of it, I don't install anything)

1. **Save this file.** If it isn't saved as `CLAUDE.md` in the current folder yet, save it there word for word, so a later session can follow it too.
2. **Python.** Check for Python 3.10+ (on Windows the command is `python`, not `python3`). If it's missing, install it yourself:
   - Windows: `winget install -e --id Python.Python.3.12`
   - macOS: `brew install python`
   - Linux: the package manager.
3. **Packages.** Run `python -m pip install kokoro-onnx soundfile faster-whisper edge-tts websockets numpy imageio-ffmpeg`.
   - imageio-ffmpeg brings its own ffmpeg, so nothing has to be on PATH.
   - On its first run, `voice.py` downloads the Kokoro voice model (about 350 MB) and a small speech-timing model (about 145 MB) into `engine/models/`. Both are free, need no account, and download once.
4. **A browser to render with.** If Google Chrome is installed, the scripts use it. If not, run `python -m pip install playwright` and `python -m playwright install chromium`; the scripts pick that up.
5. **Engine files.** Write the four engine files in section 7 into `engine/`, exactly as given. Then write `engine/lib.js` from section 2.
6. **Story and cast.** Pitch the premises (section 4) and ask me the questions in section 1. Then write `story.md` and `engine/cast.js` (section 3).
7. **Test the engine.**
   - Make `test/` with a `lines.json` of one line, and run `voice.py` on it.
   - Write a test `episode.html` that draws a background, one cast member and the captions.
   - Render stills at 0.5 s and 2 s and open them. You should see the font, the figure and the caption.
   - Then delete `test/`.

## 1. Ask me once

Ask in one message, with your defaults filled in:
- **The premise.** Default: the 10 premises from section 4 as a scored table, with the winner recommended.
- **The hero's name and look,** and the animal's.

If I answer "you pick" or don't answer, choose and keep going.

## 2. The look (write it into `engine/lib.js`)

**Frame:** a 1080x1920 canvas at 30 fps.
- Flat vector, no outlines, no texture.
- The only gradients are one soft radial vignette per shot and glows around lights.
- Each figure gets one soft shadow ellipse under it.

**Palette.** The cast's own colours are picked in step 6: saturated mid-tones that read on both beige and navy.

| use | colour |
|---|---|
| night navy | #121925 |
| slate | #273142 |
| wall beige | #d6c9b0 |
| floor brown | #6b513d |
| sky grey | #9ba7b4 |
| warm light | #f2c46d |
| danger red | #d94545 |
| warning amber | #f2b33d |
| safe green | #4cb86e |
| water blue | #3f9de0 |
| cream | #f3eee4 |
| ink | #121316 |

**Safe zones:**
- The subject sits in the middle band, y 300 to 1150, and fills it. A figure is 380 to 520 px tall in a normal shot, and bigger in close-ups.
- Captions sit at y 1300.
- Nothing important goes in the right 140 px or the bottom 280 px; the app's buttons cover them.
- Never leave half the frame empty. Frame each shot so the scene reaches down to the captions: a floor, a table, the lower floors of a building.

**Captions** are small and quiet, never the loudest thing on screen.
- **Font:** Oswald 600 from Google Fonts. Before setting `window.TOTAL`, run `await document.fonts.load('600 70px Oswald')`; a font used only on a canvas isn't loaded otherwise.
- **Size and style:** 70 px, uppercase, cream fill, an ink stroke with a 10 px line width drawn under the fill, and a soft shadow.
- **Placement:** one short line of 2 to 4 words, centred at x 540 (max width 760 px), with its middle at y 1300.
- **Chunks:**
  - A chunk prefers to break at punctuation; a comma is a soft break.
  - A chunk never ends on a small word (a, an, the, in, on, at, of, to, for, from, with, and, or, but, her, his, its, that, before, about, is).
  - A chunk never runs across two lines of the script.
- **Timing:** each chunk starts on its first word's time and stays until the next chunk starts, or until 0.35 s after its line ends, whichever comes first. The first chunk shows from frame 0.
- **Colour:** the key word of each line (from `key` in `lines.json`) gets one accent colour: red for danger, amber for warning, green for good news, blue for water or cold. Numbers stay as digits.
- **Big type instead:** a line with `"nocap": true` shows no caption. Use it for the line written in big type on black.

**Cards** are cream, rounded 28 px, with a soft shadow, 760 to 800 px wide, centred at x 540, inside y 260 to 1000.
- Each card springs in with a small overshoot on its word and stays while its line plays. The picture behind a card may cut or move while the card stays.
- Card text is at least 50 px, and pill labels at least 30 px.
- **DAY card:** a red band with the story title, a huge "DAY N", and a dark pill with the clock time.
- **RULE card:** a red pill "RULE #1", then the rule in 1 to 2 short lines of ink caps. It pops on "Rule number 1" and stays through the line that says the rule. While it waits for that line, it shows only the pill.
- **FACT card:** an amber pill "FACT", then the fact in 2 lines.
- **STAT card:** a title and 2 to 4 rows, each a label, a value and a bar. Values count on their words, and the card grows row by row.
- **WHO card:** a round avatar drawn with the cast function, the name, one line ("neighbour · 6th floor") and a trust bar.
- **CLOCK pill:** a dark pill with a red time ("3:17 AM"), centred at x 540, y 300.

**Shots.** Use at least 6 of these kinds:
- a building or place cross-section, with a dot per person and red dots for the threat;
- the window over the city;
- the door, seen from inside;
- a room;
- an object close-up;
- a peephole circle;
- a data card shot;
- one line in big type on black.

**Camera:**
- Every shot drifts: scale 1.00 to 1.06 across the shot, eased.
- Each shot gets at least one bigger move on a key word: a push in, a pan down the building, or a 0.3 s shake on an impact.

**Pace:**
- A new picture every 2 to 4 seconds.
- Something visible changes at least every 2 seconds (a card, an action, a counter, a move or a cut). A new caption alone doesn't count.
- No shot holds still for more than 4 seconds.
- A short line (like "Rule number 1.") can share a shot with the next line.

**Frame 0 is the scroll-stopper.**
- Line 1 plays over a cold open: the most dramatic moment of the video, close and big, with the hero and the threat in one shot, mid-action.
- The story then starts from the DAY card.
- The first caption is on screen from frame 0.
- Never open on black, a title card, a wide empty shot or an empty room.

**The end card** fills the middle of the frame for the last 2.5 s, after the voice ends, on near-black. It pops in with the same spring as the cards.
- The vote question, in red caps, about 90 px, in 2 lines if needed ("OPEN THE DOOR OR KEEP IT SHUT?").
- Under it, the cost of each side in cream caps, about 40 px ("OPEN: HE MAY BE BITTEN · SHUT: HE KNOWS ABOUT THE CAT").
- Under that, "COMMENT YOUR ANSWER" in cream caps, about 46 px.

**Cover mode:** when the page is opened as `episode.html#cover`, `render()` draws the cover instead of a frame. It has no captions.
- The video's strongest picture.
- A red pill with the story title.
- A 2-line title of 2 to 4 words in the top third, between y 250 and 640 so a 3:4 grid crop keeps it: line 1 in cream, line 2 in red, about 130 px, with an ink stroke.

## 3. The cast (write `engine/cast.js`)

**Each character is one function,** `drawHero(ctx, x, y, s, o)`.
- `x, y` is the point between the feet. `s` is the shot's scale; at s = 1 an adult is 420 px tall, a child about 260 px and the animal about 120 px.
- `o = {pose, facing, t, mood}`.
- Poses: stand, walk, crouch, sit (with bent knees), reach up, push (against a door), hold (an object), point and scared.
- Facing: back, left, right or front.
- `t` drives small life: breathing, a blink every 3 to 4 s, a tail flick.

**People are geometric.**
- A round head (radius about 34 px at s = 1).
- Hair as 1 to 2 shapes.
- A jacket in the character's colour, with sleeves in a darker shade.
- Dark trousers and shoes.
- Dot eyes only when facing front, and a tiny mouth line only when they speak.

**The animal** is about 120 px tall at s = 1, with big eyes that glow when it senses danger.

**The threat** is also a function in `cast.js`: a figure with a clear silhouette and a rim of red light, so it reads on dark backgrounds. Its glowing eyes may show from any side.

**One look per character.** Each character's colours, hair and proportions are set once. Every shot draws people only through these functions. A close-up of an arm or a face is the same function drawn big and cropped.

**Where the rest goes:** reusable sets, props and cards go in `lib.js`; one-off shots go in `episode.html`.

**`story.md`** holds:
- the 10 premises with their scores;
- the logline;
- the cast table (name, role, look with hex colours, voice, function);
- the rules of the threat;
- the story checklist table.

## 4. The story (this is what makes people watch to the end)

### Pitch 10 premises, keep 1
Write 10 one-line premises. Each one says:
- who: a hero with a want, a fear and a flaw;
- where: an everyday place, with a number;
- what's about to go wrong;
- the hidden twist.

Score each premise 1 to 5 on:
- **"What would I do?"** The viewer can't help picking a side.
- **A ticking clock:** water, battery, daylight, air, or time until something turns.
- **A secret:** a mystery the viewer wants solved.
- **An everyday place made deadly:** a lift, a ferry, a school, a supermarket, a tower block.
- **A hero with a want, a fear and a flaw.**

Show the table and keep the top total.

### The story checklist
Before you voice anything, print a table with one row per item below: the line number and the line that does it. Rewrite until every row is filled.

1. **The cold open is a moment, not a setup.**
   - Something is happening to someone, at a clock time.
   - For example: "At 2:40 AM, the cat wouldn't stop staring at the door." "Something scratches at the lift doors on floor 9."
2. **A real dilemma with no clean answer.**
   - Helping someone costs water, trust or safety.
   - Voice both sides: what saying yes costs and what saying no costs ("Every liter she gives is a day she loses." / "Say no, and he knows about the cat.").
3. **A reversal in the middle,** 30 to 70% through: what we believed flips.
   - The safe floor isn't safe.
   - The noise downstairs was the neighbour, not them.
   - The rescuer has a bruise.
4. **The survival detail.**
   - The RULE is earned by the scene and said like a saying ("A locked door buys you minutes, not nights.").
   - The FACT is a number that explains why the rule matters right now.
   - Use only facts you're sure are true; if you're unsure, pick another.
   - If the rule has a catch (tub water is for washing and flushing unless boiled), put it in the post caption.
5. **A cost.** Something is lost or broken, and stays lost: days of water, the door, trust, the pet's hiding place.
6. **False relief** just before the end: "At 1:50 AM, the tower goes still."
7. **The last line is an image or a line of dialogue,** never a summary: "The cat carrier is open, and it's empty." "Dad, there's someone in the lift."
8. **The vote question** on the end card splits viewers about 50/50, and both answers cost something. If the scene makes one answer obvious, add a reason for the other.
9. **No filler.** Every line raises the stakes, reveals something, or makes a choice. Cut any line that only describes.
10. **Characters say their own big lines,** in their own voice, in the first person.
11. **The 12-year-old test:** every choice makes sense with no explanation.

### What the best series of this kind do (borrow the craft, never the plots, characters or lines)
- The first line drops you into motion; the next line gives the number that makes it scary.
- Short sentences, and fragments for impact ("A guard. A nurse. Nine strangers on the stairs.").
- A list of three where the third breaks the pattern ("Ana, clear. Tomas, clear. Rui, clear." Then the dog.).
- The pet notices danger first, and the pet is also a liability: a rule bans animals, or one meow gives them away.
- One person is both a threat and someone you feel for.
- Clock times make it real: 3:17 AM, then 3:41 AM.
- The rule explains the next scene, so the viewer feels clever when it pays off.

## 5. The shape of the video (write `part1/lines.json`)

**Length:** 160 to 175 spoken words. With the narrator `am_michael` at speed 1.1 (about 160 words a minute) that's about 60 to 66 s of voice. Write about 20 to 24 lines of 2 to 12 words each, one sentence per line; a full stop inside a line adds a pause, so split the line instead.

**How the lines read:**
- Each line is one thing the viewer sees right then.
- Third person, present tense.
- A number every 2 or 3 lines.
- No gore: bites are bruises, and the danger is implied.

**The beats:**
1. **0 to 3 s, the cold open:** a moment, close and big, already under way.
2. **By 6 s, the DAY card:** the day and clock time, where and who.
3. **To about 20 s, the problem,** with a number (3 minutes of water pressure, 6 floors below), and the dilemma set up.
4. **About 20 to 30 s, the rule.** The narrator says "Rule number 1." and the RULE card pops; then the FACT card.
5. **30 to 45 s, the choice and the reversal.** The hero chooses and pays the cost; a STAT card shows the numbers change. Then what we believed flips.
6. **45 to 55 s, false relief, then the turn.** A clock time; the animal notices first; the threat arrives.
7. **The last line:** under 10 words, an image or a line of dialogue. Put `"pause": 0.6` on the line before it. The end card follows.

**`lines.json`** (`pause` is the silence after a line):
```json
{"voice": "am_michael", "speed": 1.1,
 "lines": [
  {"text": "At 2:40 AM, something scratches at the lift doors.", "pause": 0.25, "key": {"scratches": "red"}},
  {"text": "The taps cough, and go dry.", "nocap": true, "pause": 0.6},
  {"text": "Dad, there's someone in the lift.", "voice": "am_puck", "key": {"lift": "red"}}
 ]}
```
- Write each line exactly as it should be heard. Numbers stay as digits ("143 liters", "3:17 AM").
- **Narrator:** `am_michael`, a calm male voice in the pitch range these story pages use. For a woman narrator use `af_heart`.
- **Character voices:** `af_bella`, `bf_emma` (a British woman), `af_heart`, `bm_george` (a British man), `am_puck`, `am_fenrir`. Give the male characters a voice that sounds clearly different from the narrator. One voice per character, written in `story.md`.
- `voice.py` copies `key`, `nocap` and `voice` into `timing.js`.
- If Kokoro can't run on this computer, `voice.py` falls back by itself to a free online voice (edge-tts), then to the computer's built-in voice.

## 6. Make the video, step by step

1. **Write `part1/lines.json`.** Print the narration with its word count and the story checklist table. Rewrite until every row is filled. Show it to me and wait only if I asked to approve scripts.
2. **Voice.** Run `python engine/voice.py part1`. It writes `narration.wav` and `timing.js`, and prints the length, the words per minute and which voice ran.
   - If it ran over 67 s, cut words and run it again; if it ran under 58 s, lengthen the pauses on the dramatic beats.
   - If it says the voice fell back, tell me.
3. **Write `part1/episode.html`.**
   - It loads Oswald, then `../engine/lib.js`, `../engine/cast.js` and `timing.js`, then its own shots.
   - Time every shot and event from `TIMING` with a helper like `w(lineIndex, "word")`, which gives the time that word starts. Small offsets are fine (`w(4, "dies") - 0.2`); seconds typed by hand are not.
   - Set `window.TOTAL` = `TIMING.voice_end` + 2.5.
   - The page defines `window.step(dt)`, `window.render()` and `window.reset()`. `render()` draws purely from the current time.
4. **Check stills before rendering.**
   - Run `python engine/render.py part1/episode.html part1/stills --at <times>`, with one time at 0, one in the middle of every line, and one on the end card.
   - Open every still and fix what's wrong:
     - a face under the captions;
     - a subject that's small, dark or lost;
     - text over text;
     - a card off screen;
     - an empty half of the frame;
     - a pose that doesn't match the line.
     - a counter that hasn't reached its final value: take a still 0.3 s after the number is spoken, and check the card shows that exact number.
   - Repeat until every still is right.
5. **Sound.** Write `part1/sfx.json`, then run `python engine/sound.py part1 <TOTAL>`.
   - Put a sound on every action: a knock on "knocks", a thud on an impact, a hit on a reveal, a tick on counters, a hiss for the animal, a whoosh on fast moves.
   - Raise the bed under the turn and fade it on the end card.
   - A dark music bed plays by default, made in code: a low drone, a slow minor-key pad, a soft pulse, and low booms on every `hit`. It sits 16 dB under the voice and ducks under every line. Set `"music": "off"` in `sfx.json` to drop it, or `"music": {"level": -18}` for quieter.
6. **Render and encode.** Run `python engine/render.py part1/episode.html part1/frames`, then `python engine/encode.py part1`.
   - It writes `part1.mp4` (under 10 MB) and `sheet.jpg` (frame 0, then one frame every 3 s).
   - It also prints a pacing check. If it prints a PACING WARNING, fix those seconds (a new shot, a card or a move), then render and encode again.
   - If it prints STOP, the render was cut short: render again.
7. **Check.** Open `sheet.jpg`; black tiles at the very end are only padding.
   - Frame 0 must be strong.
   - No tile may look empty or broken.
   - The video must run 60 to 70 s, with sound. If it's short, lengthen the pauses on the dramatic beats.
   - Then delete `frames/`, `vo/` and `stills/`.
8. **Cover.** Run `python engine/render.py "part1/episode.html#cover" part1/cover` and save the still as `part1/cover.jpg`.
9. **Post kit.** Write `part1/post.md`:
   - the title (the cover's two lines);
   - a caption: 2 lines on what happened, then "Rule #1: ...", the fact, and the vote question ("comment OPEN or SHUT");
   - 5 hashtags;
   - a pinned comment.

Deliver `part1/part1.mp4`, `cover.jpg` and `post.md`.

## 7. Engine files (save each exactly as written)

### engine/render.py
```python
"""render.py: page -> numbered JPEG frames with headless Chrome. The page defines window.TOTAL (s),
window.step(dt), window.render(), optional window.reset().
  python render.py page.html frames              every frame at 30 fps (frames/f00000.jpg ...)
  python render.py page.html stills --at 0,3.5   only these moments (stills/at_0.00.jpg ...)
  python render.py page.html#cover cover   the page's cover mode, one still (cover/at_0.00.jpg)
  options: --fps 30 --w 1080 --h 1920 --quality 90 --port (default: any free port)     needs: pip install websockets"""
import asyncio, base64, json, os, shutil, subprocess, sys, tempfile, time, urllib.request
import websockets

def chrome_path():
    if os.environ.get("CHROME"): return os.environ["CHROME"]
    for p in [r"C:\Program Files\Google\Chrome\Application\chrome.exe", r"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe",
              "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"]:
        if os.path.exists(p): return p
    for n in ["google-chrome", "google-chrome-stable", "chromium", "chromium-browser"]:
        if shutil.which(n): return shutil.which(n)
    try:                                   # no Chrome: use Playwright's Chromium
        from playwright.sync_api import sync_playwright
        with sync_playwright() as p: path = p.chromium.executable_path
        if os.path.exists(path): return path
    except Exception: pass
    sys.exit("No Chrome found. Run: pip install playwright && python -m playwright install chromium")

def main():
    a = sys.argv[1:]
    if len(a) < 2: sys.exit(__doc__)
    page, _, frag = a[0].partition("#")
    html, out = os.path.abspath(page) + ("#" + frag if frag else ""), a[1]
    if frag and "--at" not in a: a += ["--at", "0"]
    opt = lambda k, d: type(d)(a[a.index(k) + 1]) if k in a else d
    fps, W, H, q, port = opt("--fps", 30), opt("--w", 1080), opt("--h", 1920), opt("--quality", 90), opt("--port", 0)
    if not port:                                   # a free port, so two renders can run side by side
        import socket
        with socket.socket() as sk: sk.bind(("127.0.0.1", 0)); port = sk.getsockname()[1]
    at = [float(x) for x in a[a.index("--at") + 1].split(",")] if "--at" in a else None
    os.makedirs(out, exist_ok=True)
    prof = tempfile.mkdtemp(prefix="render_chrome_")
    proc = subprocess.Popen([chrome_path(), "--headless=new", "--disable-gpu", "--hide-scrollbars", "--mute-audio",
                             "--force-device-scale-factor=1", "--allow-file-access-from-files",
                             f"--remote-debugging-port={port}", f"--user-data-dir={prof}", f"--window-size={W},{H}", "about:blank"],
                            stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    try:
        ws_url = None
        for _ in range(60):
            try:
                tabs = json.load(urllib.request.urlopen(f"http://127.0.0.1:{port}/json"))
                ws_url = [t for t in tabs if t["type"] == "page"][0]["webSocketDebuggerUrl"]; break
            except Exception: time.sleep(0.5)
        if not ws_url: sys.exit("could not connect to Chrome")
        asyncio.run(run(ws_url, html, out, fps, W, H, q, at))
    finally:
        proc.terminate()
        time.sleep(1); shutil.rmtree(prof, ignore_errors=True)

async def run(ws_url, html, out, fps, W, H, q, at):
    async with websockets.connect(ws_url, max_size=200_000_000) as ws:
        n = 0
        async def call(method, params=None):
            nonlocal n
            n += 1; my = n
            await ws.send(json.dumps({"id": my, "method": method, "params": params or {}}))
            while True:
                m = json.loads(await ws.recv())
                if m.get("id") == my:
                    if "error" in m: raise RuntimeError(f"{method}: {m['error']}")
                    return m.get("result", {})
        async def ev(expr, wait=False):
            r = await call("Runtime.evaluate", {"expression": expr, "awaitPromise": wait, "returnByValue": True})
            if r.get("exceptionDetails"):
                raise RuntimeError("page error: " + json.dumps(r["exceptionDetails"])[:800])
            return r.get("result", {}).get("value")
        await call("Emulation.setDeviceMetricsOverride", {"width": W, "height": H, "deviceScaleFactor": 1, "mobile": False})
        await call("Page.navigate", {"url": "file:///" + html.replace("\\", "/").lstrip("/")})
        for _ in range(120):
            await asyncio.sleep(0.25)
            if await ev("document.readyState==='complete' && typeof window.render==='function' && typeof window.step==='function' && window.TOTAL>0"): break
        else:
            sys.exit("page never became ready: it needs window.TOTAL, window.step(dt) and window.render()")
        await ev("document.fonts ? document.fonts.ready.then(()=>true) : true", True)
        await asyncio.sleep(0.3)
        total = await ev("window.TOTAL")
        print(f"page ready, TOTAL = {total:.2f} s", flush=True)
        times = at if at is not None else [k / fps for k in range(int(round(total * fps)))]
        await ev("window.reset && window.reset()")
        sim, t0 = 0.0, time.time()
        for k, t in enumerate(times):
            if t < sim - 1e-9:
                await ev("window.reset && window.reset()"); sim = 0.0
            steps = int(round((t - sim) * 60))
            if steps > 0:
                await ev(f"for(let i=0;i<{steps};i++) window.step(1/60); 0"); sim += steps / 60
            await ev("window.render(); 0")
            shot = await call("Page.captureScreenshot", {"format": "jpeg", "quality": q, "clip": {"x": 0, "y": 0, "width": W, "height": H, "scale": 1}})
            name = f"at_{t:.2f}.jpg" if at is not None else f"f{k:05d}.jpg"
            with open(os.path.join(out, name), "wb") as f: f.write(base64.b64decode(shot["data"]))
            if at is None and k % 150 == 0:
                print(f"frame {k}/{len(times)}  {time.time() - t0:.0f} s", flush=True)
        print(f"done: {len(times)} images in {out} ({time.time() - t0:.0f} s)")

if __name__ == "__main__":
    main()
```

### engine/voice.py
```python
"""voice.py PART_DIR : lines.json -> narration.wav + timing.js (the time of every word, for captions and animation)
Main voice: Kokoro-82M, a free open model that runs on this computer's CPU. Word times come from faster-whisper, also local.
If Kokoro can't run, it falls back by itself to edge-tts (a free online voice), then to the computer's own voice
(Windows SAPI, macOS say, Linux espeak-ng). Force one with "engine": "kokoro" | "edge" | "offline" in lines.json.
needs: pip install kokoro-onnx soundfile faster-whisper edge-tts imageio-ffmpeg numpy  (models download on the first run)"""
import asyncio, difflib, json, os, platform, re, shutil, subprocess, sys, tempfile, urllib.request, wave
import numpy as np
SR = 48000
HERE = os.path.dirname(os.path.abspath(__file__)); MODELS = os.path.join(HERE, "models")
KOKORO_URL = "https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files-v1.0/"

def ffmpeg():
    if os.environ.get("FFMPEG"): return os.environ["FFMPEG"]
    if shutil.which("ffmpeg"): return shutil.which("ffmpeg")
    import imageio_ffmpeg; return imageio_ffmpeg.get_ffmpeg_exe()
FF = ffmpeg()
part = sys.argv[1]
spec = json.load(open(os.path.join(part, "lines.json"), encoding="utf-8"))
VOICE, SPEED = spec.get("voice", "am_michael"), float(spec.get("speed", 1.1))
engine = os.environ.get("VOICE_ENGINE", spec.get("engine", "kokoro"))
os.makedirs(os.path.join(part, "vo"), exist_ok=True); os.makedirs(MODELS, exist_ok=True)

def decode(path):
    return subprocess.run([FF, "-loglevel", "error", "-i", path, "-ac", "1", "-ar", str(SR), "-f", "s16le", "-"], capture_output=True, check=True).stdout

_k = _w = None
def kokoro():
    global _k
    if _k is None:
        from kokoro_onnx import Kokoro
        for n in ("kokoro-v1.0.onnx", "voices-v1.0.bin"):
            p = os.path.join(MODELS, n)
            if not os.path.exists(p):
                print("downloading", n, "(once)"); urllib.request.urlretrieve(KOKORO_URL + n, p + ".part"); os.replace(p + ".part", p)
        _k = Kokoro(os.path.join(MODELS, "kokoro-v1.0.onnx"), os.path.join(MODELS, "voices-v1.0.bin"))
    return _k

def whisper():
    global _w
    if _w is None:
        from faster_whisper import WhisperModel, download_model
        d = os.path.join(MODELS, "whisper-base.en")
        if not os.path.exists(os.path.join(d, "model.bin")): print("downloading the timing model (once)"); download_model("base.en", output_dir=d)
        _w = WhisperModel(d, device="cpu", compute_type="int8")
    return _w

def norm(w): return re.sub(r"[^a-z0-9]", "", w.lower())
def clean(w): return w.strip(".,!?;\"()[]").rstrip(".")

def estimate(text, raw):
    """word times in proportion to word length, inside the voiced part of the line"""
    x = np.abs(np.frombuffer(raw, np.int16).astype(np.float32)); loud = np.where(x > 600)[0]
    a, b = (loud[0] / SR, loud[-1] / SR) if len(loud) else (0, len(x) / SR)
    ws = text.split(); k = [len(w) + 2 for w in ws]; tot = sum(k); out, t = [], a
    for w, n in zip(ws, k):
        d = (b - a) * n / tot; out.append((clean(w), t, d * 0.9)); t += d
    return out

def align(text, raw):
    """word times from faster-whisper, matched back to the script's own words"""
    try:
        audio = np.frombuffer(raw, np.int16).astype(np.float32)[::3] / 32768      # 48 kHz -> 16 kHz
        segs, _ = whisper().transcribe(audio, language="en", word_timestamps=True, beam_size=1, initial_prompt=text)
        heard = [(w.word, w.start, w.end) for s in segs for w in s.words]
    except Exception as e:
        print("word timing fell back to an estimate:", e); return estimate(text, raw)
    sw = text.split(); times = [None] * len(sw)
    sm = difflib.SequenceMatcher(a=[norm(w) for w in sw], b=[norm(h[0]) for h in heard], autojunk=False)
    for a, b, n in sm.get_matching_blocks():
        for j in range(n): times[a + j] = (heard[b + j][1], heard[b + j][2])
    if not any(times): return estimate(text, raw)
    end = len(raw) / 2 / SR
    i = 0
    while i < len(sw):                                   # words whisper didn't match: share the gap by length
        if times[i] is None:
            j = i
            while j < len(sw) and times[j] is None: j += 1
            t0 = times[i - 1][1] if i > 0 else max(0.0, (times[j][0] if j < len(sw) else end) - 0.3 * (j - i))
            t1 = times[j][0] if j < len(sw) else min(end, t0 + 0.35 * (j - i))
            k = [len(w) + 2 for w in sw[i:j]]; tot = sum(k); t = t0
            for m, n in zip(range(i, j), k):
                d = (t1 - t0) * n / tot; times[m] = (t, t + d); t += d
            i = j
        else: i += 1
    return [(clean(w), s, max(0.05, e - s)) for w, (s, e) in zip(sw, times)]

async def edge(text, voice, path):
    import edge_tts
    if re.match(r"^[ab][fm]_", voice): voice = "en-US-AvaNeural" if voice[1] == "f" else "en-US-AndrewNeural"
    for attempt in range(3):
        try:
            words, audio = [], bytearray()
            async for ch in edge_tts.Communicate(text, voice, rate="+5%", boundary="WordBoundary").stream():
                if ch["type"] == "audio": audio += ch["data"]
                elif ch["type"] == "WordBoundary": words.append((ch["text"], ch["offset"] / 1e7, ch["duration"] / 1e7))
            if not audio or not words: raise RuntimeError("empty answer")
            open(path, "wb").write(audio); return words
        except Exception as e:
            print("edge-tts retry", attempt + 1, e); await asyncio.sleep(2 + attempt * 2)
    raise RuntimeError("edge-tts is not answering")

def offline(text, path):
    tmp = tempfile.mkdtemp(); txt = os.path.join(tmp, "line.txt"); open(txt, "w", encoding="utf-8").write(text)
    if platform.system() == "Windows":
        ps = ("Add-Type -AssemblyName System.Speech; $s = New-Object System.Speech.Synthesis.SpeechSynthesizer; $s.Rate = 1; "
              f"$s.SetOutputToWaveFile('{path}'); $s.Speak([IO.File]::ReadAllText('{txt}')); $s.Dispose()")
        subprocess.run(["powershell", "-NoProfile", "-Command", ps], check=True)
    elif platform.system() == "Darwin": subprocess.run(["say", "-f", txt, "-o", path, "--data-format=LEI16@22050"], check=True)
    else: subprocess.run(["espeak-ng", "-f", txt, "-w", path], check=True)

async def speak(i, line):
    """one line -> (pcm at 48 kHz, word boundaries relative to that pcm)"""
    text, path = line["text"].strip(), os.path.join(part, "vo", f"line{i:02d}")
    if engine == "kokoro":
        import soundfile as sf
        a, sr = kokoro().create(text, voice=line.get("voice", VOICE), speed=float(line.get("speed", SPEED)), lang="en-us")
        sf.write(path + ".wav", a, sr); raw = decode(path + ".wav"); return raw, align(text, raw)
    if engine == "edge":
        bounds = await edge(text, line.get("voice", VOICE), path + ".mp3"); return decode(path + ".mp3"), bounds
    offline(text, path + ".wav"); raw = decode(path + ".wav"); return raw, align(text, raw)

async def main():
    global engine
    order = {"kokoro": ["kokoro", "edge", "offline"], "edge": ["edge", "offline"], "offline": ["offline"]}[engine]
    for engine in order:
        try:
            lead = spec.get("lead", 0.15)
            pcm, cursor, out = bytearray(b"\0\0" * int(lead * SR)), lead, []
            for i, line in enumerate(spec["lines"]):
                raw, bounds = await speak(i, line)
                text = line["text"].strip()
                a = max(0.0, bounds[0][1] - 0.06)                             # trim the silence around each line
                b = min(len(raw) / 2 / SR, bounds[-1][1] + bounds[-1][2] + 0.15)
                raw = raw[int(a * SR) * 2:int(b * SR) * 2]; dur = len(raw) / 2 / SR
                words = []
                for btext, off, d in bounds:                                  # one boundary can hold several words
                    ws = btext.split() or [btext]
                    for j, w in enumerate(ws):
                        s = off - a + d * j / len(ws); e = off - a + d * (j + 1) / len(ws)
                        words.append({"w": w, "s": round(cursor + s, 3), "e": round(cursor + e, 3)})
                extra = {k: v for k, v in line.items() if k not in ("text", "pause", "speed")}
                out.append({"i": i, "text": text, "start": round(cursor, 3), "end": round(cursor + dur, 3), "words": words, **extra})
                pcm += raw; gap = float(line.get("pause", 0.25))
                pcm += b"\0\0" * int(gap * SR); cursor += dur + gap
                print(f"line {i:2d}  {cursor:6.2f} s  {len(words):2d} words  {text[:60]}")
            break
        except Exception as e:
            print(f"voice '{engine}' failed ({e}); trying the next one")
    else:
        sys.exit("no voice worked")
    with wave.open(os.path.join(part, "narration.wav"), "wb") as w:
        w.setnchannels(1); w.setsampwidth(2); w.setframerate(SR); w.writeframes(bytes(pcm))
    open(os.path.join(part, "timing.js"), "w", encoding="utf-8").write(
        "window.TIMING = " + json.dumps({"voice_end": round(cursor, 3), "engine": engine, "lines": out}) + ";\n")
    n = sum(len(l["words"]) for l in out)
    print(f"narration {cursor:.2f} s, {n} words, {n / cursor * 60:.0f} words per minute, voice: {engine}")

asyncio.run(main())
```
`timing.js` sets `window.TIMING = {voice_end, engine, lines: [{i, text, start, end, words: [{w, s, e}], key?, nocap?, voice?}]}`. The word text has no punctuation, and numbers stay as written ("143", "3:17").

### engine/sound.py
```python
"""sound.py PART_DIR TOTAL_SECONDS : narration.wav + timing.js + sfx.json -> mix.wav (-14 LUFS, all sound synthesized)
sfx.json = {"bed": [{"at": 0, "level": 0.6}, {"at": "knocks@14", "level": 1.0}],
            "sfx": [{"at": "knocks@14", "type": "knock", "vol": 0.8}, {"at": 31.2, "type": "hit"}],
            "music": "on"}   (or "off", or {"level": -16}: dB under the voice while it speaks)
The music is a dark bed made here: a low drone, a slow minor-key pad, a soft pulse that quickens near the end,
and low booms on every "hit" sound. It ducks under each line, swells a little in the gaps and with the "bed" cues.
"at" is seconds, or "word@line" (first time that word is spoken in that 0-based line), optionally "+0.2" / "-0.1".
types: knock thud hit whoosh tick hiss growl creak sting heartbeat rumble glass click drip alarm
needs: pip install numpy imageio-ffmpeg"""
import json, os, re, shutil, subprocess, sys, wave
import numpy as np
def ffmpeg():
    if os.environ.get("FFMPEG"): return os.environ["FFMPEG"]
    if shutil.which("ffmpeg"): return shutil.which("ffmpeg")
    import imageio_ffmpeg; return imageio_ffmpeg.get_ffmpeg_exe()
FF = ffmpeg()
SR = 48000
part, TOTAL = sys.argv[1], float(sys.argv[2])
N = int(TOTAL * SR)
rng = np.random.default_rng(7)
timing = json.loads(re.sub(r"^\s*window\.TIMING\s*=\s*|;\s*$", "", open(os.path.join(part, "timing.js"), encoding="utf-8").read().strip()))
spec = json.load(open(os.path.join(part, "sfx.json"), encoding="utf-8")) if os.path.exists(os.path.join(part, "sfx.json")) else {}

def when(at):
    if isinstance(at, (int, float)): return float(at)
    m = re.match(r"^\s*([^@+]+)@(\d+)\s*([+-]\s*[\d.]+)?\s*$", str(at))
    if not m: return float(at)
    word, li, off = m.group(1).strip().lower(), int(m.group(2)), float((m.group(3) or "0").replace(" ", ""))
    for w in timing["lines"][li]["words"]:
        if re.sub(r"[^\w']", "", w["w"].lower()) == re.sub(r"[^\w']", "", word): return w["s"] + off
    sys.exit(f'sfx.json: "{word}" is not spoken in line {li}: {timing["lines"][li]["text"]}')

def band(x, lo, hi):
    f = np.fft.rfft(x); fr = np.fft.rfftfreq(len(x), 1 / SR)
    f[(fr < lo) | (fr > hi)] = 0
    return np.fft.irfft(f, len(x))

def env(n, a, d):
    t = np.arange(n) / SR
    return np.minimum(1, t / max(a, 1e-4)) * np.exp(-t / d)

def tone(f, n, f_end=None):
    t = np.arange(n) / SR
    fr = f if f_end is None else f * (f_end / f) ** (t / t[-1])
    return np.sin(2 * np.pi * np.cumsum(np.broadcast_to(fr, t.shape)) / SR)

def noise(n): return rng.standard_normal(n)

def make(kind):
    L = lambda s: int(s * SR)
    if kind == "knock":
        one = 0.9 * tone(120, L(0.25), 80) * env(L(0.25), 0.001, 0.05) + 0.5 * band(noise(L(0.25)), 300, 2500) * env(L(0.25), 0.0005, 0.015)
        out = np.zeros(L(1.2))
        for k in range(3): out[L(0.33 * k):L(0.33 * k) + len(one)] += one
        return out
    if kind == "thud": n = L(0.6); return tone(70, n, 40) * env(n, 0.002, 0.12) + 0.4 * band(noise(n), 60, 800) * env(n, 0.001, 0.05)
    if kind == "hit": n = L(2.2); return tone(55, n, 30) * env(n, 0.003, 0.6) + 0.35 * band(noise(n), 30, 300) * env(n, 0.002, 0.4)
    if kind == "whoosh": n = L(0.9); e = np.sin(np.linspace(0, np.pi, n)) ** 2; return 0.8 * band(noise(n), 400, 3000) * e
    if kind == "tick": n = L(0.05); return band(noise(n), 2000, 6000) * env(n, 0.0002, 0.006)
    if kind == "click": n = L(0.08); return 0.8 * tone(1800, n) * env(n, 0.0002, 0.01)
    if kind == "hiss": n = L(0.9); e = np.minimum(1, np.arange(n) / L(0.05)) * np.exp(-np.arange(n) / L(0.5)); return 0.7 * band(noise(n), 3000, 9000) * e
    if kind == "creak":
        n = L(1.1); t = np.arange(n) / SR; f = 160 + 50 * np.sin(2 * np.pi * 1.3 * t) + 15 * band(noise(n), 5, 40)
        saw = 2 * ((np.cumsum(f) / SR) % 1) - 1
        return 0.35 * band(saw, 200, 2500) * np.sin(np.linspace(0, np.pi, n))
    if kind == "sting": n = L(2.5); return 0.35 * sum(tone(f, n) for f in (220, 233.1, 311.1, 466.2)) * env(n, 0.01, 0.8)
    if kind == "heartbeat":
        one = tone(50, L(0.3), 35) * env(L(0.3), 0.004, 0.07); out = np.zeros(L(0.9))
        out[:len(one)] += one; out[L(0.28):L(0.28) + len(one)] += 0.7 * one; return out
    if kind == "rumble": n = L(2.5); return 0.9 * band(noise(n), 25, 140) * np.sin(np.linspace(0, np.pi, n))
    if kind == "glass":
        n = L(0.8); return 0.3 * sum(tone(f, n) * env(n, 0.0005, 0.1 + 0.05 * k) for k, f in enumerate((2400, 3150, 4100, 5300))) + 0.5 * band(noise(n), 3000, 10000) * env(n, 0.0005, 0.05)
    if kind == "growl":
        n = L(1.2); t = np.arange(n) / SR; f = 95 + 10 * np.sin(2 * np.pi * 3 * t)
        saw = 2 * ((np.cumsum(f) / SR) % 1) - 1
        return 0.6 * band(saw, 60, 900) * (0.6 + 0.4 * np.sin(2 * np.pi * 9 * t)) * np.sin(np.linspace(0, np.pi, n))
    if kind == "alarm":
        n = L(1.6); t = np.arange(n) / SR; gate = ((t * 4) % 1 < 0.5).astype(float)
        return 0.4 * np.sign(np.sin(2 * np.pi * 880 * t)) * gate * np.minimum(1, (n - np.arange(n)) / L(0.1))
    if kind == "drip": n = L(0.25); return 0.6 * tone(900, n, 1800) * env(n, 0.001, 0.04)
    sys.exit(f"unknown sfx type {kind}")

with wave.open(os.path.join(part, "narration.wav")) as w:
    v = np.frombuffer(w.readframes(w.getnframes()), np.int16).astype(np.float64) / 32768
voice = np.zeros(N); voice[:min(N, len(v))] = v[:N]
t = np.arange(N) / SR
drone = (np.sin(2 * np.pi * 55 * t) + 0.7 * np.sin(2 * np.pi * 82.4 * t + 1) + 0.4 * np.sin(2 * np.pi * 110.6 * t)) * (0.75 + 0.25 * np.sin(2 * np.pi * t / 7))
room = band(noise(N), 40, 400); room /= np.abs(room).max() + 1e-9
bed = 0.5 * drone / 2.1 + 0.5 * room
cues = sorted([(when(c["at"]), float(c["level"])) for c in spec.get("bed", [{"at": 0, "level": 0.7}])])
level = np.interp(t, [c[0] for c in cues] + [TOTAL], [c[1] for c in cues] + [cues[-1][1]])
level *= np.clip((TOTAL - t) / 1.5, 0, 1) * np.clip(t / 0.3, 0, 1)
venv = np.convolve(np.abs(voice), np.ones(SR // 20) / (SR // 20), mode="same")
duck = 1 - 0.55 * np.clip(venv / 0.04, 0, 1)
fx = np.zeros(N)
for c in spec.get("sfx", []):
    s = int(when(c["at"]) * SR); x = make(c["type"]); x = x / (np.abs(x).max() + 1e-9)
    e = min(N, s + len(x)); fx[s:e] += float(c.get("vol", 0.7)) * x[:e - s]
# music: a dark bed made here, never a downloaded track
mcfg = spec.get("music", "on")
music = np.zeros(N)
if mcfg not in ("off", False, None, 0):
    mdb = float(mcfg.get("level", -16)) if isinstance(mcfg, dict) else (float(mcfg) if isinstance(mcfg, (int, float)) and not isinstance(mcfg, bool) else -16.0)
    chords = [(73.42, 146.83, 174.61, 220.0), (58.27, 116.54, 146.83, 174.61), (49.0, 98.0, 116.54, 146.83), (55.0, 110.0, 130.81, 164.81)]
    seg = 8.0                                                   # D minor, B flat, G minor, A: 8 s each, slow crossfades
    for k in range(int(TOTAL // seg) + 2):
        f0 = chords[k % 4]; t0 = k * seg - 1.5; t1 = t0 + seg + 3.0
        a, b = max(0, int(t0 * SR)), min(N, int(t1 * SR))
        if a >= b: continue
        tt = np.arange(a, b) / SR
        wdw = np.sin(np.pi * np.clip((tt - t0) / (t1 - t0), 0, 1)) ** 2
        pad = np.zeros(b - a)
        for i, fr in enumerate(f0):
            for det in (-0.7, 0.7):
                ph = 2 * np.pi * (fr + det) * tt
                pad += (np.sin(ph) + 0.3 * np.sin(2 * ph) + 0.1 * np.sin(3 * ph)) * (1.0 if i == 0 else 0.5)
        music[a:b] += pad * wdw
    music = band(music, 30, 700); music /= np.abs(music).max() + 1e-9
    low = (np.sin(2 * np.pi * 36.71 * t) + 0.5 * np.sin(2 * np.pi * 73.42 * t + 0.7)) * (0.7 + 0.3 * np.sin(2 * np.pi * t / 11))
    music = 0.7 * music + 0.35 * low
    thump = tone(48, int(0.4 * SR), 34) * env(int(0.4 * SR), 0.004, 0.09)
    tp = 1.0
    while tp < TOTAL - 2.0:                                     # the pulse: 57 a minute, about 75 in the last 15 s
        i = int(tp * SR); e = min(N, i + len(thump)); music[i:e] += 0.45 * thump[:e - i]
        tp += 1.05 if tp < TOTAL - 17 else 0.8
    boom = tone(41, int(2.5 * SR), 30) * env(int(2.5 * SR), 0.01, 0.8)
    for c in spec.get("sfx", []):                               # low booms on the big beats
        if c.get("type") == "hit":
            i = int(when(c["at"]) * SR); e = min(N, i + len(boom)); music[i:e] += 0.8 * boom[:e - i]
    vslow = np.convolve(np.abs(voice), np.ones(SR // 4) / (SR // 4), mode="same")
    talking = vslow > 0.01
    mduck = 1 - 0.37 * np.clip(vslow / 0.03, 0, 1)               # about 4 dB down while a line plays
    music *= mduck * (0.75 + 0.3 * np.clip(level, 0, 1.2))       # swells with the bed cues and fades with it
    vr = np.sqrt(np.mean(voice[talking] ** 2)) if talking.any() else 0.1
    mr = np.sqrt(np.mean(music[talking] ** 2)) + 1e-9
    music *= vr * 10 ** (mdb / 20) / mr                          # mdb below the voice while it speaks
    print(f"music: {mdb:.0f} dB under the voice while it speaks")
mix = voice + 0.22 * bed * level * duck + 0.45 * fx + music
mix /= max(1.0, np.abs(mix).max() / 0.98)
raw = os.path.join(part, "mix_raw.wav")
with wave.open(raw, "wb") as w:
    w.setnchannels(1); w.setsampwidth(2); w.setframerate(SR); w.writeframes((mix * 32767).astype(np.int16).tobytes())
def measure(path):
    m = subprocess.run([FF, "-hide_banner", "-i", path, "-af", "loudnorm=print_format=json", "-f", "null", "-"], capture_output=True, text=True).stderr
    return float(json.loads(m[m.rindex("{"):m.rindex("}") + 1])["input_i"])
li = measure(raw); out = os.path.join(part, "mix.wav"); gain = -14 - li
for k in range(2):                                   # the second round corrects what the limiter took off
    if k: gain += -14 - measure(out)
    subprocess.run([FF, "-y", "-loglevel", "error", "-i", raw, "-af", f"volume={gain:.2f}dB,alimiter=limit=0.84:level=false",
                    "-ar", str(SR), "-ac", "2", out], check=True)
os.remove(raw)
print(f"mix.wav written: {TOTAL:.2f} s, {measure(out):.1f} LUFS (from {li:.1f})")
```

### engine/encode.py
```python
"""encode.py FOLDER : FOLDER/frames + FOLDER/mix.wav -> FOLDER/<folder>.mp4 (H.264 + AAC, under 10 MB),
FOLDER/sheet.jpg (frame 0, then one frame every 3 s) and a pacing check (cuts per 10 s, longest still).
needs: pip install numpy imageio-ffmpeg"""
import os, re, shutil, subprocess, sys
import numpy as np
def ffmpeg():
    if os.environ.get("FFMPEG"): return os.environ["FFMPEG"]
    if shutil.which("ffmpeg"): return shutil.which("ffmpeg")
    import imageio_ffmpeg; return imageio_ffmpeg.get_ffmpeg_exe()
FF = ffmpeg()
part = os.path.normpath(os.path.abspath(sys.argv[1])); name = os.path.basename(part) + ".mp4"
out = os.path.join(part, name)
import wave
with wave.open(os.path.join(part, "mix.wav")) as w: need = int(w.getnframes() / w.getframerate() * 30)
have = len([n for n in os.listdir(os.path.join(part, "frames")) if n.endswith(".jpg")])
if have < need - 3: sys.exit(f"STOP: frames/ has {have} frames but the sound needs {need}: the render stopped early, render again")
for crf in (22, 24, 26, 28, 30):
    subprocess.run([FF, "-y", "-loglevel", "error", "-framerate", "30", "-i", os.path.join(part, "frames", "f%05d.jpg"),
                    "-i", os.path.join(part, "mix.wav"), "-vf", "scale=out_range=tv:out_color_matrix=bt709,format=yuv420p",
                    "-c:v", "libx264", "-preset", "slow", "-crf", str(crf), "-maxrate", "1100k", "-bufsize", "2200k",
                    "-x264-params", "colorprim=bt709:transfer=bt709:colormatrix=bt709:range=tv",
                    "-c:a", "aac", "-b:a", "128k", "-shortest", "-movflags", "+faststart", out], check=True)
    mb = os.path.getsize(out) / 1e6
    if mb < 10: break
rows = -(-len(os.listdir(os.path.join(part, "frames"))) // 540)
subprocess.run([FF, "-y", "-loglevel", "error", "-i", out, "-vf", rf"select='not(mod(n\,90))',scale=270:-1,tile=6x{rows}",
                "-frames:v", "1", "-fps_mode", "vfr", os.path.join(part, "sheet.jpg")], check=True)
info = subprocess.run([FF, "-hide_banner", "-i", out, "-af", "ebur128", "-f", "null", "-"], capture_output=True, text=True).stderr
dur = re.search(r"Duration: (\d+):(\d+):([\d.]+)", info); secs = int(dur.group(2)) * 60 + float(dur.group(3))
lufs = re.findall(r"I:\s+(-?[\d.]+) LUFS", info)
print(f"{name}: {secs:.1f} s, {mb:.1f} MB (crf {crf}), loudness {lufs[-1] if lufs else '?'} LUFS, contact sheet: sheet.jpg")
# pacing: how often the picture cuts, and where nothing on screen moves at all
raw = subprocess.run([FF, "-loglevel", "error", "-i", out, "-vf", "fps=10,scale=54:96,format=gray", "-f", "rawvideo", "-"], capture_output=True).stdout
f = np.frombuffer(raw, np.uint8).reshape(-1, 96, 54).astype(np.float32)
d = np.abs(np.diff(f, axis=0)).mean(axis=(1, 2))
cuts = []
for i in np.where(d > 14)[0]:
    if not cuts or (i + 1) / 10 - cuts[-1] > 0.25: cuts.append((i + 1) / 10)
stills, start = [], None
for i, s in enumerate(np.append(d < 0.35, False)):
    if s and start is None: start = i
    if not s and start is not None:
        if (i - start) / 10 > 2.5: stills.append(f"{start / 10:.1f}-{i / 10:.1f} s")
        start = None
gaps = [f"{a:.1f}-{b:.1f} s" for a, b in zip([0.0] + cuts, cuts + [secs]) if b - a > 5.0]
per10 = len(cuts) / secs * 10
print(f"pacing: {per10:.1f} cuts per 10 s (aim for 2.5 or more)")
if per10 < 2.5 or stills or gaps:
    if stills: print("PACING WARNING: nothing moves at " + ", ".join(stills) + ": add a move or a card there")
    if gaps: print("PACING WARNING: no cut for over 5 s at " + ", ".join(gaps) + ": cut to a new picture there")
    if per10 < 2.5: print("PACING WARNING: too few cuts overall: give more lines their own shot")
else:
    print("pacing OK")
```

## 8. Gotchas
- `render()` must give the same picture for the same time, every time: no unseeded `Math.random()`, `Date.now()`, `requestAnimationFrame` or CSS animations.
- Time every event from `TIMING`. If a line changes, run `voice.py` again and everything moves with it.
- Draw captions last, on top of everything. Keep faces out of y 1240 to 1360.
- A dark figure on a dark background disappears. Give it a rim of light, or a lighter wall behind it.
- Cards and captions never cover each other.
- Every counter (a STAT card value, a clock, a percentage) reaches its final value no later than the word that says it, and stays on screen at least 0.5 s after that. Start the count on the beat that causes it, and keep the card up until the next line starts. A card that fades out mid-count shows a wrong number.
- If the video runs long, cut words or shorten pauses. Keep the voice `speed` between 1.0 and 1.1.
- If you can't open local pages in a browser tool, use `render.py --at` and open the stills.
- Never use `ctx.filter` (blur): it crashes long renders. Make soft shadows from 3 or 4 stacked ellipses at low opacity, and glows from radial gradients.
- Oswald loads from Google Fonts, so rendering needs an internet connection.
- The animal's sound: `hiss` for a cat, `growl` for a dog; `alarm` for a bell or a siren.

## 9. Optional: Part 2

Only when I say "make part 2" (and tell you how the vote went):
1. Read `story.md`, `part1/lines.json` and Part 1's ending.
2. Make `part2/` exactly as above, with these changes:
   - **The cold open** answers the vote within 8 s, with a twist.
   - **The cast:** reuse `engine/cast.js` and `lib.js` unchanged. You may add poses or props at the end of the file, but never change a colour or a shape.
   - **Continuity:** the numbers and losses from Part 1 carry over (the water left, the broken door).
   - **The rule** is Rule #2, with a new fact.
   - **The ending** is a new vote question.
3. Add Part 2 to `story.md`.

Post Part 2 the next day at the same hour as Part 1.
