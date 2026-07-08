# Implementation Plan: Cat Petting Clicker

## Overview

Build a single `index.html` file that delivers the full cat-petting clicker experience using HTML5, Tailwind CSS CDN, and inline Vanilla JavaScript. Implementation progresses from the static shell outward to interactive logic, then to each reaction type, then to polish and edge-case handling.

All code lives in one file. The implementation language is **Vanilla JavaScript** (browser, no build step).

---

## Tasks

- [x] 1. Create the HTML shell with Tailwind CDN, custom config, and Google Fonts
  - Create `index.html` with `<!DOCTYPE html>`, `<head>`, and `<body>` structure
  - Add the `tailwind.config` script block **before** the Tailwind CDN `<script src>` tag — define `wiggle` and `zzz-fade` keyframes and `animate-wiggle` / `animate-zzz` animation utilities
  - Add the Tailwind CDN `<script src="https://cdn.tailwindcss.com">` tag
  - Add the Google Fonts `<link>` for "Quicksand"; add CSS `font-family` fallback to `sans-serif` in a `<style>` block
  - Set `<body>` to use the pastel gradient background (`#FFDEE9` → `#B5FFFC`)
  - _Requirements: 7.1, 7.2, 9.1, 9.2, 9.3, 9.4_

- [x] 2. Build the glassmorphism card layout and static UI elements
  - Add the centered frosted-glass card container (`backdrop-blur-md bg-white/30 border border-white/40 rounded-3xl shadow-xl`)
  - Add the `<h1>` title: "🐱 Cat Petter"
  - Add the Pet Counter `<p id="pet-count">` with initial text "Total Pets: 0 🐈"
  - Add the Aura Score `<p id="aura-score">` with initial text "✨ Aura: 0"
  - Add the cat+overlay wrapper `<div class="relative ...">` and inner `<div id="overlay-container">`
  - Add the hint text paragraph "Click the cat to pet it!"
  - _Requirements: 6.1, 6.2, 7.1, 7.3_

- [x] 3. Add the SVG cat illustration
  - Add the inline SVG element with `id="cat"` inside the cat+overlay wrapper
  - Implement 8 shape primitives: body ellipse, head circle, left ear polygon, right ear polygon, left eye ellipse, right eye ellipse, nose polygon, tail path
  - Apply `cursor-pointer`, `select-none`, `aspect-square`, and `transition-transform duration-300` Tailwind classes
  - Add `aria-label="Clickable cat — click to pet!"` for accessibility
  - Verify the cat is visually centered and renders between 128px–256px on a standard viewport
  - _Requirements: 7.3, 7.4, 7.5_

- [x] 4. Implement the UI Updater module
  - Inside `<script>` at the bottom of `<body>`, define the `UIUpdater` object with `petCount`, `auraScore`, `petCountEl`, and `auraScoreEl` references
  - Implement `UIUpdater.init(petCountEl, auraScoreEl)` to cache DOM references
  - Implement `UIUpdater.incrementPetCount()`: `petCount++`, set `petCountEl.textContent = \`Total Pets: \${petCount} 🐈\``
  - Implement `UIUpdater.addAura(amount)`: `auraScore += amount`, set `auraScoreEl.textContent = \`✨ Aura: \${auraScore}\``
  - Call `UIUpdater.init(...)` on DOM ready
  - _Requirements: 6.1, 6.3, 2.3_

  - [ ]* 4.1 Write property test for pet counter monotonic increment
    - **Property 1: Pet counter monotonically increases**
    - For any N calls to `UIUpdater.incrementPetCount()`, `UIUpdater.petCount` equals N
    - **Validates: Requirements 1.3, 6.1, 6.3**

  - [ ]* 4.2 Write property test for aura score accumulation
    - **Property 3: Aura score accumulates correctly on Happy**
    - For any N calls to `UIUpdater.addAura(5)`, `UIUpdater.auraScore` equals 5×N
    - **Validates: Requirements 2.3**

- [x] 5. Implement the Sound Manager module
  - Define the `SoundManager` object with a `sounds` map
  - Implement `SoundManager.init()`: pre-create `new Audio('sounds/pop.mp3')`, `new Audio('sounds/purr.mp3')`, `new Audio('sounds/zoom.mp3')` and store in `sounds`; wrap in `try/catch` — errors are swallowed silently
  - Implement `SoundManager.play(key)`: `try { sounds[key].currentTime = 0; sounds[key].play(); } catch(e) {}`
  - For Purr: expose `SoundManager.playPurr()` — set `loop = true`, play, then `setTimeout(() => { audio.pause(); audio.loop = false; }, 2000)` — all inside `try/catch`
  - Call `SoundManager.init()` on DOM ready
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 3.3_

  - [ ]* 5.1 Write property test for sound failure isolation
    - **Property 6: Sound failure does not interrupt reaction or counter**
    - For any reaction type, if `SoundManager.play()` throws, reaction still executes and petCount still increments
    - **Validates: Requirements 1.5, 3.4, 8.2**

- [x] 6. Implement the Overlay module
  - Define the `Overlay` object with `containerEl` reference
  - Implement `Overlay.init(containerEl)` to cache the overlay container `<div>`
  - Implement `Overlay.showText(text, displayMs, fadeMs)`: create `<span>`, set `textContent` (not innerHTML), append to container, apply fade-in CSS transition, `setTimeout` to start fade-out after `displayMs`, then `remove()` after `fadeMs` more
  - Implement `Overlay.showZzz()`: create 3 spans with `animate-zzz` class; schedule each at 0ms, 500ms, 1000ms; after 2000ms fade out all simultaneously over 500ms then remove from DOM
  - Implement `Overlay.showSpeedLines()`: show a `<span>` with "💨" text; remove after 1000ms with 300ms fade-out
  - Implement `Overlay.clearAll()`: remove all child elements from `containerEl`
  - _Requirements: 2.2, 4.2, 4.3, 5.2_

  - [ ]* 6.1 Write property test for overlay content matching reaction
    - **Property 8: Overlay text matches reaction type**
    - For any Happy trigger, overlay contains "YAY! 😆 (+5 Aura)"; for Sleeping, exactly 3 Zzz elements appear; for Zoomies, at least one 💨 element appears
    - **Validates: Requirements 2.2, 4.2, 5.2**

- [-] 7. Implement the Animation Controller module
  - Define the `AnimationController` object with `catEl` and `happyTimer` properties
  - Implement `init(catEl)` to cache the cat SVG reference
  - Implement `applyHappy()`: if `happyTimer` exists, `clearTimeout(happyTimer)` (don't add duplicate class); else `catEl.classList.add('animate-bounce')`; set `happyTimer = setTimeout(() => removeHappy(), 1500)`
  - Implement `removeHappy()`: `catEl.classList.remove('animate-bounce'); happyTimer = null`
  - Implement `applyPurring()`: `catEl.classList.add('animate-pulse')`; `setTimeout(() => catEl.classList.remove('animate-pulse'), 2000)`
  - Implement `applySleeping()`: `catEl.classList.add('scale-90', 'rotate-6')`; `setTimeout(() => catEl.classList.remove('scale-90', 'rotate-6'), 2500)`
  - Implement `applyZoomies()`: within 50ms `catEl.classList.add('animate-wiggle')`; `setTimeout(() => catEl.classList.remove('animate-wiggle'), 400)`
  - Call `AnimationController.init(...)` on DOM ready
  - _Requirements: 2.1, 2.4, 2.5, 3.2, 3.5, 4.1, 4.4, 5.1, 5.4_

  - [ ]* 7.1 Write property test for reaction class assignment vs roll
    - **Property 7: Custom animation class assignment matches roll ranges exactly**
    - For any integer r in [1,100]: `selectReaction(r)` returns 'happy' iff r ∈ [1,40], 'purring' iff r ∈ [41,70], 'sleeping' iff r ∈ [71,90], 'zoomies' iff r ∈ [91,100]
    - **Validates: Requirements 2.1, 3.2, 4.1, 5.1, 5.6**

  - [ ]* 7.2 Write property test for Happy double-click idempotence
    - **Property 5: Happy double-click idempotence on CSS classes**
    - For any N rapid happy clicks while bounce active, `animate-bounce` appears in classList exactly once
    - **Validates: Requirements 2.5**

- [ ] 8. Implement the Reaction Engine module and wire everything together
  - Define the `ReactionEngine` object with `isActive`, `activeTimeout`
  - Implement `selectReaction(roll)`: `if roll <= 40 return 'happy'; if roll <= 70 return 'purring'; if roll <= 90 return 'sleeping'; return 'zoomies'`
  - Implement `executeReaction(type)`:
    - `'happy'`: `AnimationController.applyHappy(); Overlay.showText('YAY! 😆 (+5 Aura)', 1500, 300); UIUpdater.addAura(5); SoundManager.play('pet')`
    - `'purring'`: set `isActive = true`; `AnimationController.applyPurring(); Overlay.showText('~ purring ~', 2000, 300); SoundManager.playPurr()`; after 2000ms, `isActive = false`
    - `'sleeping'`: set `isActive = true`; `AnimationController.applySleeping(); Overlay.showZzz(); SoundManager.play('pet')`; after 2500ms, `isActive = false`
    - `'zoomies'`: `AnimationController.applyZoomies(); Overlay.showSpeedLines(); SoundManager.play('zoom')`
  - Implement `handlePet()`: if `isActive` return early; `UIUpdater.incrementPetCount()`; `const roll = Math.floor(Math.random() * 100) + 1`; `executeReaction(selectReaction(roll))`
  - Attach `catEl.addEventListener('click', () => ReactionEngine.handlePet())`
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 3.6, 4.5_

  - [ ]* 8.1 Write property test for reaction probability coverage
    - **Property 2: Reaction probabilities are mutually exclusive and exhaustive**
    - For all integers r in [1,100], `selectReaction(r)` returns exactly one valid reaction string; the union of all ranges covers [1,100] with no gaps or overlaps
    - **Validates: Requirements 1.1, 2.1, 3.2, 4.1, 5.1, 5.6**

  - [ ]* 8.2 Write property test for blocking guard
    - **Property 4: Blocking guard prevents overlapping Purring/Sleeping reactions**
    - For any click while `isActive === true`, `handlePet()` returns early — petCount does not increment, no new reaction fires, no CSS classes are added
    - **Validates: Requirements 3.6, 4.5**

- [~] 9. Checkpoint — Verify core interaction loop
  - Ensure all tests pass, ask the user if questions arise.
  - Manual smoke tests: click cat, verify Pet Counter increments, verify each reaction type fires (force roll values), verify Aura Score increases only on Happy.

- [ ] 10. Handle edge cases and polish
  - [~] 10.1 Add large pet count edge case handling
    - Verify `UIUpdater.incrementPetCount()` correctly displays counts > 999999 (JavaScript integers handle this natively; confirm no `toLocaleString` rounding or truncation is used)
    - Confirm `textContent` is used (not `innerHTML`) for all overlay and counter text — XSS prevention
    - _Requirements: 6.4, security_

  - [ ]* 10.2 Write edge case test for large pet count
    - Set `UIUpdater.petCount = 999999`, call `incrementPetCount()` once, assert `petCountEl.textContent === 'Total Pets: 1000000 🐈'`
    - **Validates: Requirements 6.4**

  - [~] 10.3 Verify Tailwind custom animation config is present and correct
    - Confirm `tailwind.config` block precedes the Tailwind CDN `<script>` tag in the HTML
    - Confirm `wiggle` keyframe uses correct `translateX` values as specified
    - Confirm `zzz-fade` keyframe uses correct `opacity` / `translateY` values
    - Confirm `animate-wiggle` maps to `wiggle 0.4s` and `animate-zzz` maps to `zzz-fade 1s`
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

  - [~] 10.4 Verify idle state on load
    - Confirm no animation classes are present on `#cat` before any click event
    - Confirm Pet Counter shows "Total Pets: 0 🐈" and Aura shows "✨ Aura: 0" on load
    - _Requirements: 6.2, 7.5_

- [~] 11. Final checkpoint — Full integration verification
  - Ensure all tests pass, ask the user if questions arise.
  - Verify the full reaction set manually: Happy (force roll 1), Purring (force roll 50), Sleeping (force roll 80), Zoomies (force roll 95)
  - Verify sound failure is silent: rename/remove a sound file, reload, confirm no console exceptions and reactions still fire
  - Verify Purring/Sleeping blocking: click rapidly during a reaction, confirm extra clicks are ignored

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP build
- The app is a single `index.html` file — no bundler, no npm, no build step
- Sound files (`sounds/pop.mp3`, `sounds/purr.mp3`, `sounds/zoom.mp3`) are optional assets; the app degrades gracefully without them
- Tailwind CDN `tailwind.config` must be set **before** the CDN script loads — order matters
- All DOM text mutations use `textContent`, never `innerHTML`, to avoid XSS
- Property tests reference design.md properties 1–8 for traceability
