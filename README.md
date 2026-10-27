# FORRA RESIDENCE — hero animation

Open `index.html` through any local web server (e.g. `npx serve .`). The page loads GSAP 3.12.5 and ScrollTrigger from cdnjs and the fonts from Google Fonts.

## Sequence

All times are counted from the film's last frame. The film plays at 1.12× (`FILM_RATE` in `index.html`; the footage is unchanged, 4.5 s instead of 5.04 s). The camera moves until the very last frame, so the title starts exactly on that frame: it is detected with `requestVideoFrameCallback` (Chrome, Edge, Safari), with an animation-frame check as a fallback, and never waits for the browser's `ended` event. The film sits beneath the headlines and the house cut-out; the cut-out is hidden during the film and appears on the last frame, which is the same picture, so the title is occluded by the roof from its first instant. On «Ещё раз» the film rewinds out of sight and dissolves in from its first frame.

| # | Segment | Starts | What happens |
|---|---------|--------|--------------|
| 1 | `film` | 0.00 s | On the last frame the house cut-out appears over the identical film frame; the film then fades out underneath, over the matching still (0.45 s). |
| 2 | `forra` | 0.00 s | Drafting reveal: a fine centre line draws out with end ticks (0.3 s), then opens into two boundaries (0.8 s, power3 in-out) with vertical extension lines at the word's edges, a measuring scale ticking along the lower guide, a dimension line with architectural slashes above, and corner crosshairs locking on. FORRA is clipped to exactly the space between the boundaries. The drawing then retracts and fades (0.45 s) and is removed. The word itself never moves or scales. |
| 3 | `residence` | 0.19 s | The same drafting reveal a beat later. |
| 4 | `text` | 1.28 s | The slogan rises line by line through its mask; the services list and the location simply fade in. |
| 5 | `numbers` | 2.38 s | 12 DEC.2027, then 200 M², then 80.000 $. Each digit spins on a 3-D dial, slows down, stops on its value, and the neighbouring digits fade out. |
| 6 | `details` | 3.45 s | The separators draw, the stat labels fade in, then the logo, navigation, SCHEDULE A VIEWING, REQUEST A CALL and the phone button appear. |
| 7 | `map` | 4.66 s | The floor-plan card unmasks upward, then the plan, then the caption fades in, then the four indicator bars. |

Total: about 5.7 s from the film's last frame.

**Pace.** All text and interface reveals run at `TEXT_SPEED = 1.45` (in `index.html`, next to `START`): every segment's timeline plays 1.45× faster and the start times are compressed by the same factor, so the order and rhythm are unchanged. The durations quoted in the table rows above are at the original pace (divide by 1.45). Set `TEXT_SPEED` to 1 for the earlier, slower version. The film itself is not affected.

## «Ещё раз» (replay)

When the intro finishes, an outline pill «Ещё раз» appears at the bottom centre of the hero. Clicking it scrolls back to the top, resets the room tour, hides the interface and plays everything again from the first frame of the house film. The button hides while the sequence plays and returns when it ends. From the console: `FORRA.playAll()`.

## Window sizes

The page is one full-window hero. It always fills the browser window exactly (no scroll needed to see it, no bars or extra layers around it).

* **Desktop (900 px and wider):** the 1920 × 1080 Figma composition. Text, buttons, statistics and card keep their design size and only scale down on smaller screens (a 1440 × 900 laptop shows them at 75 %). They are anchored to their corners: the left groups stay left, RESIDENCE, the services list, the location and the card stay right, and the statistics, card and «Ещё раз» stay at the bottom. Extra room in a larger window goes between those groups.
* **Mobile (narrower than 900 px):** a dedicated 390 × 844 composition of the same elements, anchored the same way.
* **Photo:** the still, the house cut-out and the film share one transform, so they always stay registered. They cover the window without distortion; on very wide windows the crop comes mostly from the ground, so the roof keeps its designed overlap with RESIDENCE.

All of this is set by `fit()` in `index.html`.

## Replaying one animation on its own

Every segment builds its own timeline and its own cleanup, so it can run inside the full sequence or by itself over the finished page.

* Open the page with `#segments` at the end of the address to show a small player bar with one button per segment.
* From the console: `FORRA.play('forra')`, `FORRA.play('numbers')`, … or `FORRA.playAll()`. The list of names is in `FORRA.segments`.
* Start times live in the `START` object. The drop values live in `SEGMENTS`: `fall` (fall duration), `squash` (height at landing as a fraction of the final height — keep it between 0.5 and 0.7), `press` (squash duration) and `expand` (expansion duration).

## Scroll room tour

After the intro, scrolling pins the hero and plays the room tour (scrubbed with the scrollbar, so scrolling back reverses it):

1. The floor plan grows from the card to full screen, shown whole at full resolution (`assets/floorplan-hd.jpg`, 1916 × 958). The card's four indicator bars become the tour progress: plan, living room, bedroom, bathroom.
2. Every photo, the plan included, is held for the same time (`HOLD`) and changes to the next with the same length of transition (`CHANGE`). Both are in `index.html`, in scroll units.
3. Each change uses a transition chosen automatically (see below).
4. The tour holds on the last room, then the photo layer fades away and the plain hero screen remains, with the card back in its corner.

Add rooms by adding `<img class="tour-img" src="…" data-name="…">` inside `#tourFrame`. The scroll length grows with the number of rooms.

## How the transition is chosen

For every pair of consecutive photos the page measures how alike the two rooms look:

* **Layout:** both photos are shrunk to a 16 × 9 grid of brightness values, and the average difference between the grids is taken.
* **Tone:** the difference between the two photos' average colours.

`similarity = 0.6 × layout + 0.4 × tone`, on a scale from 0 (unrelated) to 1 (identical).

* **similarity ≥ 0.88 → room tour (walk-through).** The current photo pushes in toward its brightest opening (the window or light source, found automatically as the brightest 3 × 3 block of the grid) while the next room settles in from a slight zoom around its own brightest point. It feels like walking through to the next room. It is used when light and composition continue, so the movement reads as one space.
* **similarity < 0.88 → dissolve.** A soft cross-dissolve with a gentle drift. It is used when the light or framing changes a lot, where a walk-through would look like a jump.

With the current photos: plan → living room scores 0.421 (dissolve), living room → bedroom 0.902 (walk-through) and bedroom → bathroom 0.859 (dissolve). The threshold is `TOUR_THRESHOLD` in `index.html`; the measured choices are listed in `FORRA.tourModes`.

## Accessibility

With "reduce motion" enabled in the operating system, the film and intro are skipped and the finished hero shows immediately. The scroll tour still works, because it only moves when the visitor scrolls.

## Cursor and hover states

**Cursor.** With a mouse or trackpad the cursor is a small line-drawn house (14 × 15 px): the FORRA A-frame (gable roof, walls, door) in the same thin white line as the drafting marks; the roof apex is the pointing tip. Difference blending keeps it visible on light areas. Touch devices keep their normal behaviour.

| Element | Hover |
|---|---|
| Menu pills | the house flows into the pill's shape as a white fill behind the label, which turns ink |
| SCHEDULE A VIEWING | sliding doors: the white face splits at the centre and both halves slide apart (0.45 s), revealing ink; a white copy of the label is masked exactly inside the opening, so the label is white wherever the doors are open and ink wherever they are closed |
| REQUEST A CALL + phone | ringing phone: the circle turns ink, the receiver turns white and gives a short ring-shake, thin rings pulse outward while hovered; the REQUEST A CALL pill shows a thin ink frame drawn inward around its inside. Clicking either part gives the pair a short buzz (a 0.34 s shake of about 2 px) and a 25 ms vibration on phones that support it |
| Logo | dims slightly |

Over the menu and the buttons the house cursor shrinks to 80 % and returns to full size when you leave. Keyboard focus shows the same states plus a white outline; with reduced motion the changes are instant and the ringing does not loop.

## About Project section

Below the hero (menu item ABOUT PROJECT scrolls to it, `#about`). A light "drawing sheet" after the dark hero, read sideways.

* **Horizontal scroll.** On desktop (900 px and wider) the section pins to the window and the ordinary vertical scroll moves a horizontal track of panels (GSAP ScrollTrigger, scrubbed, so scrolling back reverses it). The scroll length equals the width of the track. On phones, with reduced motion, or without GSAP, the same track becomes a native sideways scroller with snap points (swipe, trackpad, or the arrow keys once it has focus).
* **Panels:** 1 — the broken title ABOUT / PR◯JECT (140 px at 1920, smaller than FORRA) with the lede and a small model shot beside ABOUT; 2–5 — the four steps (01 Site 3 weeks, 02 Architecture 12 iterations, 03 Materials 24 reviewed, 04 Living space 200 m²), each with one small photo; 6 — the location, where the rule ends with a tick.
* **The rule.** One thin line with fine ticks runs through every panel like a dimension line. Each step has a bullet on it; the bullet fills in the accent colour once the step passes 55 % of the window.
* **Visible scroll cues.** The section's top bar shows a counter (00 / 04 → 04 / 04) and a «Scroll →» cue with a slowly nudging arrow («Swipe» on phones); a progress line along the bottom fills as the track moves.
* **Images:** natural proportions (width set, height automatic, never cropped or stretched), alternating heights near the rule, with a gentle ±6 % parallax drift on desktop. All are placeholders, marked `data-temp`; replace `src` and `alt` when the final photos arrive.
* **Adding a panel:** copy an `<article class="ap ap-step">…</article>` block inside `.about-track`; the scroll length and counter adapt (update the "/ 04" total in the top bar).
* **Motion:** track scrub 0.8 s with linear easing (the scroll itself is the easing); title words wipe in left to right (0.9 s), text and pictures rise 14 px and fade in (0.8 s); bullets fill in 0.4 s. Reduced motion: no pinning, no reveals, no arrow nudge.
* **Colours:** paper `#ecebe6`, ink `#292a2d` (12:1), muted text `#66645f` (4.95:1), accent `#8f5a2c` (4.8:1) — all above WCAG AA.

### Update — real photos, free scrolling, line drawing

* **Scrolling is free from the first moment.** The first scroll intent during the intro (wheel, swipe, scroll keys or a menu link) finishes the intro at once on its final frame and the page scrolls on (`skipIntro()` / `flushScroll()` in `index.html`).
* **Photos:** six of the fifteen supplied pictures, cut from the collage and upscaled 4× (EDSR super-resolution), saved as 1200 px WebP in `assets/about-*.webp`: house (intro), mountains (01 Site), floor plan (02 Architecture), materials board (03 Materials), living room (04 Living space), terrace pool (location). The rest were left out on purpose — one picture per panel.
* **Image layout:** one picture per point, all identical — 4:3, the same width and height on screen, and the files are all 1200 × 900 px. No captions. The floor plan sits whole on its own paper background so it fits the same frame.
* **Title:** PROJECT is spelled out in full; the small round house photo now sits after the word as an accent.
* **Line drawing:** the faint rule is drawn over in ink as the track moves (the drawing front sits at 55 % of the window; a bullet fills when the line reaches it). Each picture appears with a thin frame that draws around it, the photo opens inside, then the frame fades. The section's top rule draws in from the left, and the step figures count up once.
* **Round photo after PROJECT:** a two-step reveal. Once the word has drawn, fine 45° drafting lines sweep across the circle (0.6 s); only when they are complete does the photo fade in over them (0.7 s), and then the lines fade away. CSS only (`.about-o .o-hatch`), triggered by the same reveal class as the title; with reduced motion the photo simply shows.

### Update — calmer title panel, strip reveals, the opening of the next section

* **Title panel:** only ABOUT / PROJECT, the short lede and the line; the round photo and the extra house picture were removed.
* **Strip reveal for pictures** (About and Our other projects): a thin drafting frame draws round the picture, then seven paper strips pull away one after another, alternately up and down, while the photo settles from a slight zoom — about 2.3 s per picture. Helper: `dressPicture()` in `index.html`.
* **Pause, then the next section:** at the end of the About track the page rests — one second after the last picture has fully appeared (`PAUSE` in `index.html`) — then the last picture (the terrace) grows out of its frame to fill the window (1.5 s, expo in-out), the shade comes up and the title of the next section rises line by line. Scrolling back folds it away; scrolling on quickly opens it at once. Phones and reduced motion: the next section simply follows, with its own full-bleed opening image.
* **Our other projects** (`#projects`, the menu's OUR PROJECTS item): four cards that turn into place one after another (0.6 s each, about their left edge) and, on hover or keyboard focus, lift, tilt at most a few degrees towards the pointer and ease the photo in. **Titles and pictures are placeholders** (`data-temp`): replace «Project 02…05», the «Soon» labels and `assets/prj-02…05.webp`. With reduced motion the cards are simply shown.

### Update — corner opening and the architect's sketch

* **The last picture is no longer enlarged.** After the rest (one second once the last picture has fully appeared) a sheet of paper opens from the **bottom-right corner** as a quarter circle, its edge drawn as a thin arc, and the title «Our other projects» rises line by line (`buildOpening()`).
* **Our other projects is now a scroll-driven sketch** (desktop, pinned for 1.8 window heights): a line draws in from the left (0–14 % of the scroll), splits into two branches (14–22 %), which settle into two level guide lines that run on to the right (22–94 %). Each project grows from its guide the moment the line reaches it — standing on the upper line or hanging from the lower one, joined by a short tick and a dot — with the strip reveal; the counter goes 00 → 04. Scrolling back undraws the lines and closes the projects again. No line ever crosses a picture. Code: `prjSketch` in `index.html` (SVG paths with `pathLength=1`, stroke-dashoffset driven by ScrollTrigger scrub).
* **Hover:** a project tile grows a little (`scale(1.04)`, 0.45 s); keyboard focus does the same.
* **Phones / reduced motion:** a plain one- or two-column list; each project opens with the strips as it scrolls in.

### Update — vertical project flow, contour reveal, dark opening

* **Picture reveal everywhere (About + projects):** the contour draws all the way round the picture first (1.1 s), then the whole photo appears in one smooth fade (0.7 s); no strips. CSS `.ph.has-reveal`.
* **Our other projects scrolls vertically** (no pinning, no horizontal movement): one continuous line runs down the middle of the section (down the left edge on phones) and grows with the scroll; the drawing front sits at 62 % of the window. When it reaches a project, a level branch and a node reach out, then the picture's contour draws and the photo appears — one project after another, alternating left and right. Scrolling back undraws the line and closes the projects. Code: `prjSketch` in `index.html`.
* **Corner opening is now dark and clearly visible:** a near-black sheet (`#121314`) grows from the bottom-right corner over 1.8 s behind a warm drawn arc; on it the title (left) and an index of the four projects (right, rules drawn one by one), so the landing screen is never empty. The projects section continues on the same dark background with light text — contrast 15.6:1 (text), 8.6:1 (secondary), 7.6:1 (accent).

### Update — scroll-only corner opening, richer sketch

* **Corner opening is driven only by the scroll** (no automatic playback): after the About track the page rests for 0.3 of a window height of scrolling, then the dark sheet opens from the bottom-right corner over **1.4 window heights** of scrolling, slow enough to follow; scrolling back closes it. `REST` / `OPEN` in `index.html`.
* **Decoration at the top of the sheet:** a drawn dimension line with ticks every 48 px, slashes at both ends, a quarter arc in the right corner and a short guide line — drawn as the sheet opens (`.ns-deco`, built in `buildOpening()`).
* **The projects sketch** now has several kinds of line, all drawn by the scroll front: the main line with a fine scale beside it, a second line that twists around it, a long parallel construction line, and at each project a branch that turns out on a quarter arc, a 45° construction line overshooting into the empty side, extension lines past the picture's corners and a dimension line along its outer edge. The main line, its scale and the twisting line are clipped exactly at the drawing front.

### Update — straight lines only, lines that gain strength

* **Our other projects:** one continuous route of straight segments only — down the main line to a project, sideways out to it (a square node marks the turn), down along its edge, back sideways to the main line, down to the next project, and so on. No curves and no diagonals anywhere in the sketch; the extra marks (scale ticks, extension lines past the pictures' corners, a dimension line with square end ticks) are straight too and never run ahead of the route's tip.
* **Gradient lines:** the sketch is stroked with a vertical gradient — faint at the top (28 %), full strength at the bottom — and the About ink line goes from faint to full ink along its length.
* **Fix:** picture contours no longer show stray corner marks before they draw (the contour is now a pixel-true rectangle).

### Update — straight corner, repository

* The decoration on the dark corner sheet no longer bends on a quarter arc: the top dimension line turns the corner **straight down** as a second, shorter dimension line with the same ticks and the same end slash.
* The project lives in the GitHub repository `bodya-dis/forra` (branch `main`).

### Trial — Our other projects, version B (`#projectsB`)

A second, trial version of the projects section, placed **below** the current one (which is unchanged). Delete the block marked `TRIAL — OUR OTHER PROJECTS, VERSION B` (HTML, CSS and the `pv2` script) to drop it.

* **Look:** a vertical stack of project photos in 3-D on a dark background. The photo in the centre lies flat and sharp; the ones above and below tilt away (up to 30°), shrink, blur (up to 6 px) and darken.
* **Motion:** scroll-driven only — the section pins and about 70 % of a window height of scrolling moves the next project into the centre (scrubbed 0.9 s), so every change can be followed. The big number and the project name on the left change with the centred project.
* **Margins:** small letter-spaced words (Site / Light / Material / Form, FORRA Studio, More than a house, Drawn built lived in), thin rules, and the line «© 2026 FORRA Residence. Project pictures are placeholders.»
* **Content:** six placeholder projects (`assets/prj-02…07.webp`, `data-temp`) — the pictures come from the image set you supplied; replace them and the names when the real projects are ready.
* **Phones:** the same stack, wider; the number and name sit under it. Reduced motion / no GSAP: a plain list of flat photos.

### Trial — Our other projects, version C (`#projectsC`) — replaces version B

The 3-D stack trial (version B) has been removed. In its place, still **below** the unchanged main projects section:

* **Motion (scroll only, pinned for 2.2 window heights):** the projects come in from the left **in single file**, small, one a step behind the other; each walks **half way round a circle** (a faint dashed ring marks it) and, as soon as its short walk is over, peels off and settles into its place in the **rows** (3 × 2 on desktop, 2 × 3 on phones), growing to full size; its number and name fade in last. The counter goes 00 → 06. Scrolling back reverses everything.
* Spacing of the file: `GAP`; length of the settle: `SET` (in the `pv3` script).
* Placeholders: «Project 02…07» and `assets/prj-02…07.webp` (`data-temp`), plus «© 2026 FORRA Residence. Project pictures are placeholders.»
* Reduced motion / no GSAP: the rows straight away.
* To drop the trial, delete the block marked `TRIAL — OUR OTHER PROJECTS, VERSION C` (HTML, CSS and the `pv3` script).
* **Update (trial C):** background drawing — a dash-dotted circle (the one the projects walk on) with dash-dotted axes and a faint inner circle, plus a very faint outline of a gabled house in the bottom-right corner (desktop). Pictures are wider on desktop (16:10) and the rows fill the width, so there is little empty space at the sides. Each project has an invented name, a one-line description and a status tag — «Already completed» (green dot), «Will be ready soon» (warm dot) or «Will finish in 2027/2028» (blue dot). **Hover or keyboard focus** on a settled project turns over its photos every 1.4 s (three per project, with progress dashes) and raises the description over the picture; leaving shows the first photo again. All names, texts and pictures are placeholders (`data-temp`).

### Update — page order, «Our projects», architectural background

* **Removed:** the previous «Our other projects» section with the drawn line and branches (and its `prjSketch` script).
* **Order now:** About Project → the corner sheet, which fully covers the window → **Our projects** (`#projects`, the menu's OUR PROJECTS item): the single-file walk round the circle that settles into rows, with statuses and the hover photo carousel (formerly trial C — see above for details).
* **Heading:** «Our projects» (on the corner sheet and on the section).
* **Background:** both the corner sheet and Our projects sit on an architectural background — the floor plan turned into a dark blueprint (light lines on near-black, `assets/arch-bg.jpg`), fixed to the window so the sheet and the section meet without a seam; only the faint dash-dotted walking circle is drawn over it. The sheet's index lists the project names.

### Update — calmer corner sheet, plain background, side notes

* **Corner sheet:** the downward branch is gone — only the top dimension line with ticks and end slashes remains. The title and index were removed from the sheet (so «Our projects» appears once, on the section itself). Instead the sheet carries a statement — «Every house we draw begins with its site. …» — that **lights up word by word as you scroll**, from dim to full strength; small labels «About our work / ©2026» at the top and spaced notes along the bottom (Site · Light · Material · Form, Carpathians / Hirne Village, Drawn built lived in).
* **Background:** the blueprint image was removed (`arch-bg.jpg` deleted); the sheet and Our projects are plain near-black `#161718`, with only the faint dash-dotted walking circle.
* **Our projects side decoration (desktop):** letter-spaced notes set vertically along both edges (FORRA Studio — Carpathians; Site · Light · Material · Form — Est. 2026) and «Drawn · built · lived in» at the bottom right; the rows keep clear of them.

### Update — modern project list, designer touch on the sheet

* **Corner sheet:** keeps its drawn dimension line and the statement that lights up word by word; adds a small photo of the house set into the indent of the first line, with a hairline caption «FORRA Residence / 003» (it opens upward as the sheet arrives).
* **Our projects, restyled** (same content, same single-file walk into rows): heading in light Poppins 300 on a hairline, small «/ 03» label; square photos without rounding or shadow; each caption on a hairline — a small «·02» index, the name in large light type, the status after «↳» in small capitals.

### New section — Dimensions (`#dimensions`, after Our projects)

3-D motion typography in the spirit of the supplied reference («3D Movie with Stacked Dimensions»), on the paper background, scroll-driven and pinned for 3.2 window heights:

1. **A cube of type turns in space** — six faces, alternately ink and paper, set with the site's own words and facts: «We don't design places to live.», «We design houses.», «200 m²» (with a repeated print), «12 Dec. 2027 / 80.000 $», «FORRA RESIDENCE» repeated, «Site · Light · Material · Form».
2. **The camera moves in** — the cube grows past the frame and fades; a photo on a sheet arrives from depth, tilted, and settles into a three-quarter view; the slogan «We don't design places to live — we design houses —» slides across in huge type (difference blend: dark on paper, light over the photo).

Screen readers get the slogan and facts as plain text. Reduced motion / no GSAP: the cube shown still at an angle, the photo and slogan below. Timeline in the `dim3` script.

### Update — feedback round, and the Location section

* **Corner sheet:** the photo lost its small captions and moved to the top right, clear of the statement; the statement's lines are a little tighter (line-height 1.08) and kept to 74 % of the width.
* **Our projects:** the dash-dotted walking circle is gone. Before the projects arrive, a straight-line architectural drawing fills the frame — a plan outline with inner walls and door gaps, window marks, axis lines and dimension lines with ticks; it draws in over the first 12 % of the scroll and clears, stroke by stroke, as the projects come in.
* **Dimensions, rewritten:** six cube faces, no phrase repeated — «A house shaped by the slope.», «Glass to the peaks, stone to the ground.», «200 m²» (open rooms, terraces), «12 Dec. 2027» (price 80.000 $), «Drawn by hand, measured on site.», «Timber, stone and glass.». After the cube, the sheet cycles through five pictures — the house, the site, the plan, the materials, the rooms — with a counter; the matching word darkens in a line of large light type that sits on the paper **below** the picture (never over it).
* **New: Location (`#location-globe`, / 05):** in the spirit of the second reference — a line-drawn globe (meridians and parallels, orthographic, drawn on a canvas) turns with the scroll inside a technical frame (corner brackets, dashed rules, ticks); a tilted ring of type runs round it («FORRA RESIDENCE · CARPATHIANS · HIRNE VILLAGE · UKRAINE»); a small square marks the site; five page dots step through five short lines under the globe; halfway the sheet inverts from paper to ink. No coordinates are shown, as none were supplied. Pinned for 2.6 window heights.

### Update — hero: merged with the newer published version; call button, scroll cue

* **Built on the newer hero that was published from another session** (one-line FORRA RESIDENCE title, glass panel under the numbers, plain-text menu with a growing underline, sharper 3840 px images `assets/bg-3840.webp` / `assets/house-3840.webp`). The glass panel and the dial animation on the numbers are kept.
* **REQUEST A CALL back exactly as drawn in the design:** a white pill (203 × 54) and, next to it, a separate white circle (54) with the phone — with its earlier hover (ink circle, ringing, inner frame on the pill) and the buzz on click.
* **Menu hover is text-only:** no fill and no button look — the label turns white and a thin line grows under it.
* **«Ещё раз» removed.** Above the floor-plan card a cue «SCROLL DOWN — to open the rooms» with a thin animated line appears when the intro ends and fades once the page is scrolled. Replay remains available from the console (`FORRA.playAll()`).
