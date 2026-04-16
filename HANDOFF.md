# Ooblish — Development Handoff

## What the app is

**Ooblish** is a browser-based English ↔ Ooblish translator. Ooblish is an invented language
created by **Kayd Phelan** — the rule is simple: every vowel (A, E, I, O, U, and Y when acting
as a vowel) is replaced by the syllable **"oob"**. The app lives in a single file: `index.html`.

The lore: Ooblish is the native language of **Oobsloobnd** (Ooblish for "island"), a remote
island in the Tropical Pacific Ocean.

---

## Repository

**GitHub:** `cpPDX/Ooblish`  
**Active branch:** `claude/review-index-file-7spwu`  
**Base:** `main` (all changes on this branch are pending PR)

---

## What was built in this session

Everything lives in `index.html`. The app started as a single-page translator with broken CSS
(en-dashes instead of `--`, a `- {}` universal selector, stray markdown fences). Those were
fixed in earlier PRs. This session added:

### 1. Multi-page navigation
Three pages: **Translate**, **Challenge**, **About**. Nav tabs in the header switch between
them. `showPage(name)` handles visibility.

### 2. Challenge mode
An Ooblish → English guessing game.
- A random Ooblish phrase is displayed; the player types the English original.
- Scoring: +1 per correct answer, streak badge after 2+ in a row.
- Matching is loose: punctuation is stripped and both the guess and answer are run through
  `toOoblish()` before comparison, so minor spelling differences pass.
- 20 built-in phrases in `CHALLENGE_PHRASES[]`.
- ElevenLabs / browser TTS button to hear the phrase spoken.
- Enter key advances (check answer → next challenge).

### 3. About page
- Origin lore: Oobsloobnd island, Tropical Pacific Ocean.
- Full rules grid (A/E/I/O/U/Y → oob, with examples).
- IP credit section for Kayd Phelan (inventor of the language).
- UI credit footer.

### 4. English TTS (speak button on source panel)
A `▶` button was added to the English input panel so users can hear the original text spoken
aloud. Uses ElevenLabs with natural voice settings if a key is set, otherwise browser
`SpeechSynthesis`.

### 5. Stop toggle on all speak buttons
All three speak buttons (English, Ooblish, Challenge) now toggle between `▶` (play) and
`■` (stop). Clicking while playing calls `stopAudio()`. Two globals track state:
`currentSpeakBtn` and `currentSpeakPill`.

### 6. ElevenLabs TTS tuning
After several iterations:
- **Model:** `eleven_multilingual_v2` (highest quality, most natural prosody — was `eleven_turbo_v2_5`)
- **Ooblish voice settings:** `stability: 0.5, similarity_boost: 0.6, style: 0.1, use_speaker_boost: true`
- **English voice settings:** `stability: 0.5, similarity_boost: 0.75, style: 0.2, use_speaker_boost: true`
- **`toOoblishTts(text)`:** runs `toOoblish()` then inserts hyphens between adjacent `oob`
  tokens (`/oob(?=oob)/g` → `oob-`). This only fires for words with consecutive vowels
  (e.g. "audio", "beautiful") and gives ElevenLabs clear syllable boundaries without the
  word-break pauses that space-separation caused.
- English TTS sends raw text (no preprocessing).

### 7. Voices: American English only
Reduced from 5 mixed voices (British + American) to 2 American English voices:
- **Aria** `9BWtsMINqrJLrRacOk9x` — American female, warm (default)
- **Roger** `CwhRBWXzGAHq8TQ4Fs17` — American male, confident

### 8. Logo
Replaced the `🫧` emoji (bubbles) with a custom inline SVG: a speech bubble outline with a
wave path inside, rendered in the accent green (`#4ade80`). Crisp at any resolution.

---

## Architecture

Everything is one file. No build step, no dependencies beyond two Google Fonts.

```
index.html
├── <style>          CSS custom properties + all component styles
├── <body>
│   ├── .bg-orbs     Ambient background animation (fixed, z-index 0)
│   ├── <header>     Logo (SVG) + nav tabs + ElevenLabs key pill
│   ├── .key-drawer  Collapsible API key input + voice selector
│   ├── #page-translator
│   │   ├── .lang-bar-wrap   English → Ooblish direction indicator
│   │   └── <main>           Two panels (src/dst) + action row
│   ├── #page-challenge      Guessing game card
│   ├── #page-about          Lore + rules + IP credit
│   ├── <footer>
│   └── .toast               Notification overlay
└── <script>         All JS inline
    ├── isVowelAt()       Safari-safe vowel detection (no lookbehind)
    ├── toOoblish()       Core translation
    ├── toOoblishTts()    TTS variant: adds hyphens between adjacent oobs
    ├── showPage()        Page navigation
    ├── Key/voice mgmt    toggleDrawer, saveKey, selectVoice
    ├── Translator        onInput, translate, setOutput, clearAll, copyOoblish
    ├── Speech            stopAudio, browserSpeak, speakText, speak,
    │                     speakEnglish, speakChallenge
    └── Challenge         initChallenge, loadChallenge, checkAnswer,
                          nextChallenge
```

---

## ElevenLabs integration

The app uses the **ElevenLabs REST API** client-side. The key is entered by the user and
never leaves the browser (stored in a JS variable, not localStorage).

```
POST https://api.elevenlabs.io/v1/text-to-speech/{voiceId}
Headers: xi-api-key, Content-Type: application/json, Accept: audio/mpeg
Body:    { text, model_id: "eleven_multilingual_v2", voice_settings: {...} }
```

If no key is set, the app falls back to `window.SpeechSynthesis` (browser built-in).

---

## Known limitations / future ideas

- **TTS still imperfect for Ooblish** — neural TTS was not trained on constructed phonetics.
  The hyphen injection and model choice help, but voice cloning (recording ~1 min of someone
  reading Ooblish and uploading to ElevenLabs) would give the most natural result.
- **No persistence** — the API key and selected voice reset on page reload. Could store in
  `localStorage`.
- **Challenge phrases are hardcoded** — easy to extend `CHALLENGE_PHRASES[]`.
- **No reverse translator** — Ooblish → English is ambiguous (multiple English words map to
  the same Ooblish output) but a best-effort reverse could be fun.
