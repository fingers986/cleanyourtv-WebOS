# Volume Bar Themes

CleanYourTV includes a system-wide custom volume OSD (On-Screen Display) that overlays over all apps, inputs, HDMI sources and live TV without taking remote or mouse focus.

It is off by default: LG's own volume bar stays unless you choose a theme in the manager app, **Experimental** > **Volume bar**. It was tried on webOS 5 only.

**What it changes on the TV.** A theme other than System default switches LG's own bar off (LG's setting `com.webos.surfacemanager.volumeOSD`). The value the TV had is saved in `backup/volume-osd.orig` first and put back when you choose **System default**, by `undo-home.sh` and by `uninstall.sh` (`scripts/volume-osd-restore.sh`). A TV that does not report the setting is left alone: LG's bar stays and the custom one is not shown.

---

## Supported Themes

1. **Pill (`pill`)**
   - Vertical slender pill (62px x 280px) docked to the right bezel.
   - White liquid fill with inverted difference-blended speaker icon and numeric volume.

2. **Capsule (`capsule`)**
   - Horizontal capsule (66px x 520px) at top center.
   - Integrated speaker glyph, sound output label, 14px thick level bar, and numeric readout.

3. **Dial (`dial`)**
   - Circular radial gauge (150px x 150px) docked on the right bezel.
   - Frosted acrylic dial with responsive stroke-dasharray arc and prominent central numeric readout.

4. **System default (`system`)**
   - Native LG webOS volume OSD. Disables the custom overlay and puts LG's bar back as the TV had it.

---

## How to Add a New Theme (Contributor Guide)

Adding a new volume theme is straightforward and requires only 3 steps:

### Step 1: Register your theme in `notify/app/index.html`

In `notify/app/index.html`, locate the `VOLUME_THEMES` object and add your theme:

```javascript
VOLUME_THEMES['my-theme'] = {
  name: 'My New Theme',
  render: function (s) {
    // s: { volume: 0-100, pct: 0-100, mute: boolean, output: string, device: string, icon: string }
    var val = s.mute ? 0 : s.pct;
    return '<div class="vol-my-theme' + (s.mute ? ' mute' : '') + '">' +
      '<div class="my-device">' + s.device + '</div>' +
      '<div class="my-bar"><div class="my-fill" style="width:' + val + '%;"></div></div>' +
      '<div class="my-num">' + (s.mute ? 'MUTE' : s.volume) + '</div>' +
    '</div>';
  },
  // Optional: provide update(el, s) for 60fps in-place DOM updates during rapid volume holding
  update: function (el, s) {
    var val = s.mute ? 0 : s.pct;
    if (s.mute) el.classList.add('mute');
    else el.classList.remove('mute');
    var fill = el.querySelector('.my-fill');
    if (fill) fill.style.width = val + '%';
    var num = el.querySelector('.my-num');
    if (num) num.textContent = s.mute ? 'MUTE' : String(s.volume);
    var dev = el.querySelector('.my-device');
    if (dev && dev.textContent !== s.device) dev.textContent = s.device;
  }
};
```

### Step 2: Add scoped CSS rules in `notify/app/index.html`

Add your styles inside `<style>` in `notify/app/index.html`:

```css
.vol-my-theme {
  position: absolute;
  right: 50px;
  top: 50%;
  transform: translateY(-50%) translateX(100px);
  opacity: 0;
  transition: transform 0.22s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.18s ease;
}
#vol-card.on .vol-my-theme {
  transform: translateY(-50%) translateX(0);
  opacity: 1;
}
```

> **Rules for CSS in webOS:**
> - Never use CSS `gap` on flex containers. Use sibling margins (`* + *` or `margin-left/right`) instead.
> - Avoid features newer than Chromium 68 (no `aspect-ratio`, no `inset`, no `:is()` / `:where()`).

### Step 3: Add to the TV Settings Menu in `manager/src/js/menu.js`

In `manager/src/js/menu.js`, locate `view === 'volumebar'` and add your theme to the `themes` list:

```javascript
{ id: 'my-theme', label: tr('My New Theme'), sub: tr('Short description of theme') }
```

Also add your theme name to `function volumeSub(theme)` in `manager/src/js/menu.js`.

After updating, run:
```sh
node tools/build-pages.js
node tools/i18n.js --sync
```
and provide translations in `i18n/*.json` for the new strings.

---

## Technical Details

- **Overlay Window:** `com.cleanyourtv.notify` runs as a transparent `topui` web window.
- **Input Neutrality:** Calls `setInputRegion([{x:0,y:0,width:0,height:0}])` and `setKeyMask([])` so all remote and pointer events pass straight through to whatever app or video is currently playing.
- **Audio Service:** `helper/audio.js` subscribes to `com.webos.service.audio/master/getVolume` for output and mute state. When the output is `external_arc`, it reads the displayed level from `com.webos.service.eim/getArcVolume` and sends it to the overlay via `notify.show({ kind: 'volume', ... })`. Other outputs continue to use the master volume. ARC reporting depends on the TV and soundbar's HDMI-CEC support.
- **System Fallback:** Setting `volumeTheme` to `'system'` sets `com.webos.surfacemanager.volumeOSD` back to the value saved in `backup/volume-osd.orig`, restoring LG's volume graphics.

