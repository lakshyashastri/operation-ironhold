# Operation Ironhold

A complete first-person shooter that runs in a browser tab. One HTML file, no build step,
no package manager, and not a single image, audio or model file on disk.

> Forked from **[StarKnightt/operation-ironhold](https://github.com/StarKnightt/operation-ironhold)**
> — MIT, © StarKnightt. The original game and the five prompts that produced it are entirely
> upstream's work. This fork doubles the size of the map, adds a configurable garrison size,
> removes the round timer, generates patrol routes instead of hand-placing them, adds a third
> air jump, carries more reserve ammunition, and adds a second playable character who does not
> use guns.

**[Play the original here](https://starknightt.github.io/operation-ironhold/)**

![Gameplay](screenshots/gameplay.jpg)

A garrison holds a container yard in Sector 7. Clear it. You pick how many are down there —
anywhere from ten to a hundred — and there is no clock.

You also pick *who walks in*. **Operator** is the original game: four weapons, 150 effective
health, and a fight you can lose. **Dark Lord** is the same yard, the same squad and the same
props, with a wand instead of a rifle and nine spells instead of four guns.

The entire game — renderer setup, world generation, weapon handling, enemy AI, spellcasting,
audio synthesis, post-processing and HUD — is about 8,300 lines of JavaScript and CSS inlined
into a single 406 KB `index.html`. The only external resource is three.js r128, pulled
from a CDN. Everything else is generated at runtime.

The original was built by an AI agent from five prompts, recorded verbatim in
[PROMPTS.md](PROMPTS.md).

## Running it

Open `index.html` in any desktop browser. That is the whole installation process.

If you would rather serve it:

```bash
python -m http.server 8123
```

Then visit `http://127.0.0.1:8123/index.html`.

Click the page to start. The click is what grants pointer lock, so the game cannot capture
your mouse until you ask it to. Press `Esc` to release the pointer and pause, then click
the overlay to resume.

Requires a desktop browser with WebGL and Pointer Lock. There is no touch input, so it
will not play on a phone.

## Controls

### Operator

| Input | Action |
| --- | --- |
| `W` `A` `S` `D` | Move |
| Mouse | Look |
| Left click | Fire |
| Right click | Toggle aim down sights |
| `R` | Reload |
| `Shift` | Sprint, or hold breath while scoped |
| `Ctrl` | Crouch |
| `Space` | Jump; tap twice more in the air for two further hops; hold near a ledge to mantle |
| `1` `2` `3` `4` | M4 carbine / KS-12 pump / P-9 sidearm / SR-7 Longbow |
| `V` | Toggle the rifle between full auto and semi |
| Mouse wheel | Cycle weapons |
| `Esc` | Pause |

### Dark Lord

| Input | Action |
| --- | --- |
| `W` `A` `S` `D` | Move |
| Mouse | Look |
| Left click | Cast the selected curse |
| Right click | Hold for Protego |
| `1` `2` `3` `4` | Avada Kedavra / Bombarda Maxima / Crucio / Sectumsempra |
| Mouse wheel | Cycle curses |
| `Q` | Tap to apparate; **hold** to travel as smoke |
| `E` | Expelliarmus |
| `F` | Levicorpus |
| `C` | Petrificus Totalus |
| `R` | Homenum Revelio |
| `Space` | Jump; two air hops; **hold** after they are spent to hover |
| `Esc` | Pause |

## The round

Eliminate every hostile. The slider on the start screen sets how many, from 10 to 100, and
defaults to 25; the value applies to the next round, including `REDEPLOY`. There is no time
limit — the round ends when the last man is down, or when one of them puts you down.

As the Operator you start with 100 health and 50 armor, where armor absorbs half of incoming
damage until it is gone. Neither regenerates and there are no pickups, so the 150 you start
with is the whole budget however many hostiles you chose.

Hostiles hold fire for the first three seconds so you can get your bearings, and a combat
director caps how many may shoot at any one moment. The rest keep manoeuvring. That single
constraint is what keeps a firefight readable instead of collapsing into crossfire you
cannot answer.

For the Operator that cap is two, exactly as the game shipped. It was briefly raised for
both characters and that was a mistake: it measurably changed the original game, roughly
doubling incoming damage against a large garrison, which is not something a second campaign
should do to the first. The Dark Lord scales instead — seven at ten hostiles up to ten at a
hundred — because with 250 health, a shield and regeneration, two shooters leave him not
merely safe but unaware anything is happening.

Dying ends the round. The report screen shows eliminations, headshots, accuracy and time
survived — the stopwatch still runs, it just is not a limit any more. The Dark Lord's report
counts spells cast and kills by Unforgivable Curse instead of headshots and accuracy, because
a curse does not miss and the accuracy number would always read 100%.

## Weapons

| Slot | Weapon | Magazine | Reserve | Behaviour |
| --- | --- | --- | --- | --- |
| 1 | M4 Carbine | 30 | 480 | 760 rpm full auto, `V` switches to semi |
| 2 | KS-12 Pump | 8 | 128 | Nine pellets per shell, pump between shots |
| 3 | P-9 Sidearm | 15 | 240 | 430 rpm semi automatic, fastest draw |
| 4 | SR-7 Longbow | 5 | 65 | Bolt action, a body hit kills |

Each carries a viewmodel built from primitives with gloved hands, sways against mouse
movement, bobs in time with footsteps, dips off screen to reload and ejects brass that
bounces off the concrete. Muzzle flash is a sprite backed by a real point light that the
bloom pass picks up. The rifle climbs on a fixed recoil pattern rather than random scatter,
so the spray is something you can learn to hold down.

## Aiming

![Sniper scope](screenshots/scope.jpg)

Right click toggles ADS. Every weapon takes exactly 0.2s to settle in or out, tightens its
spread, and costs 40% of your movement speed while held.

| Weapon | FOV | Zoom | Sight |
| --- | --- | --- | --- |
| M4 Carbine | 75 to 38 | 2.2x | Red dot on tinted glass |
| KS-12 Pump | 75 to 60 | 1.3x | Bead raised clear of the heat shield |
| P-9 Sidearm | 75 to 50 | 1.7x | Three-dot irons with an open notch |
| SR-7 Longbow | 75 to 15 | 5.8x | Scope overlay, mil-dot reticle |

The sniper is the one that changes the screen. Scoping swaps the crosshair for a full
overlay and hides the weapon behind the glass, which the composite shader veils the way
real optics do. The view drifts on a slow two-sine sway; holding `Shift` steadies it for
three seconds, after which you have to release and take another breath. Mouse sensitivity
scales with the zoom so your aim stays consistent on screen. The bolt takes 1.5s to work,
which makes a miss genuinely expensive.

## Movement

Container tops, crates, barrel stacks, the wrecked flatbed and every other solid prop are
walkable. A jump clears roughly 1.2m, and two further air hops add about 0.9m and 0.7m, so a
fully chained triple jump tops out around 2.7m. Holding `Space`
while airborne near a ledge up to about 1.95m above your feet pulls you up over it, which is
what gets you onto a container roof from flat ground and onto the sniper deck without the
ramp.

## Dark Lord

The second character on the start screen. The map does not change, the props do not change and
the squad does not change — only who is holding the other end of the fight.

He carries 250 health, no armour, a shield, and health that comes back on its own a few
seconds after the last round that hit him. This is deliberate and it is not subtle: he is
meant to be the most dangerous thing in the yard, and standing motionless in the open against
a hundred armed men takes about ten seconds to kill him. Against ten or twenty-five it does
not kill him at all.

| Key | Spell | What it does |
| --- | --- | --- |
| `1` | Avada Kedavra | A green beam that kills whatever it touches. No falloff, no headshot multiplier, no armour — none of it applies. Brings the entire yard down on you. |
| `2` | Bombarda Maxima | A blast at the aim point: falloff damage in a 6.5m radius, and everyone inside it gets thrown. The containers do not move; the static world is merged and instanced, so there is nothing there to break. |
| `3` | Crucio | Held, not tapped. He drops his rifle, goes down and screams — and the screaming carries, on the same propagation the gunfire alert uses. Holding a man under it is how you call the rest of the yard to a spot of your choosing. |
| `4` | Sectumsempra | Seven hitscan samples fanned across the crosshair. Heavy damage plus a bleed that keeps running, and it takes its victims apart — see below. |
| `Q` | Apparition | Tap for an instant hop up to 26m. Hold and you become a low black streak, steerable with the mouse, for as long as you keep holding — and the yard slows to a third of speed around you while you do. |
| `E` | Expelliarmus | Takes the rifle off him and throws it. He is then unarmed, and reacts accordingly. |
| `F` | Levicorpus | Hoists him 2.5m into the air, upside down, swinging, for five seconds — then drops him, which hurts. |
| `C` | Petrificus Totalus | Six seconds rigid. Cheapest thing on the list and one of the most useful. |
| `R` | Homenum Revelio | An expanding shell that ticks once per soldier it reaches and lights him as a silhouette through the containers for five seconds. |
| RMB | Protego | Held. Blocks everything from a 150-degree frontal arc. |

**Protego is a shield, not a dome.** Every source of damage in the game already funnels through
one function, so the shield is a single arc test inside it — which means it would cover anything
added later without being told about it. The one thing it cannot do is stop a round in flight,
because enemy fire is hitscan and resolved instantly; you get the flare on the shield and the
damage simply never lands, which reads fine.

**The Horcrux.** Once per round, dying does not end the round. He comes apart, reforms
somewhere else on the map with 35 health, and every man in the yard feels it. At 250 health
with a shield you will rarely see it, which is the point.

**Apparition can never land you somewhere illegal, and it can put you on a roof.** The
destination is clamped inside the fence, resolved to the surface actually being aimed at,
searched outward for standing room *at that height*, then shoved out of anything still
overlapping. Deliberately not routed through the reachability check the patrol generator uses:
that one flood-fills from the player's spawn at ground level, so no container roof or warehouse
deck is in it, and delegating to it meant a hop aimed at a roof got snapped into the lane beside
it and blinking while stood on one dropped you off. The test exists to stop a soldier being
stranded where he cannot walk out, which is not a hazard for someone with a glide and a triple
jump. Verified at zero bad landings — outside the fence, inside a prop, sunk into the ground, or
still stuck after a full second of settling physics — across 2,500 blinks, 400 smoke flights and
300 Horcrux relocations, with 15% of blinks landing above ground level and the highest at 8.2m.

**Sectumsempra dismembers.** Head, both arms and both legs come off and tumble away along the
axis of the cut, with the torso spinning off separately. The pieces are the soldier's *own* rig
nodes, not stand-ins: `collapseRig` merges meshes by material *within* each group, so head, each
upper arm and each thigh survive as separate nodes carrying their own merged geometry.
Reparenting one to the scene costs no new geometry, no new material and — measured — 36 extra
draw calls across a hundred and eight flying pieces, because those meshes were already being
drawn individually. A man opened up by the curse who survives and bleeds out ten seconds later
still comes apart, because the mark rides on him rather than on the cast.

Two things about that were subtle enough to get wrong first time round, and did. A rig node's
origin is the *joint* it hangs from — a thigh's origin is the hip, and the boot is 0.93m below
it — so a piece left on that origin pinwheels about one end like a hinge instead of tumbling,
and a ground test against that origin buries the entire limb before it registers a landing. Both
are fixed by moving each node onto the piece's own centre as it detaches and pushing its children
back by the same amount, so the geometry does not move but the pivot does. Resting pieces now sit
within about 10cm of the surface under them, against a limb-length error before.

**Nothing winds up.** Every spell casts on the frame you press the button; the cooldown is
the whole cost. Avada Kedavra originally demanded half a second on the button before it would
fire, on the theory that the signature curse should feel deliberate. In the hand it just felt
like the spell was arguing with you.

**The smoke is its own system.** Apparition lays down a rope of black along the *whole* path —
origin to destination for an instant hop, and continuously behind you in flight — rather than
puffing at the ends. It needed a dedicated particle system: an order of magnitude more live
particles than blood and dust together, a per-particle seed so two thousand overlapping sprites
do not read as two thousand identical discs, and its own integrator for the swirl that turns a
cone of puffs into a curling tendril.

Getting it to look like the films took three passes and the failures are instructive. Large
sprites laid sparsely are a grey wall up close and a string of gaps at range. Small sprites at
the same spacing are a swarm of flies. Density and size have to be chosen together so
consecutive sprites overlap at the spacing used. And the noise that gives each puff its interior
structure has to be weighted to the *rim*: applied evenly it punches holes through the middle
too, so no amount of stacking ever reaches opacity and the rope stays a grey speckle. Weighted
outward, the body goes solid and only the silhouette frays — which is what the films actually
look like, a dense dark mass shedding particulate along its edge. It costs 0.09ms a frame at
two thousand live particles and no extra draw calls at all, being one `Points` system.

**Apparition dilates the yard, not you.** Holding `Q` slows the world — the squad, their
tracers, their brass, the dust — to 34% while you keep real time. That split is why the speed
could come down from 30 m/s to 15 and still feel fast: relative to the men you are passing you
are moving faster than before, while the input under your hand stays responsive. There is no
duration limit; you rematerialise when you let go. Your own cooldowns and regeneration run on
the yard's dilated clock rather than yours, because otherwise holding the key would have been
a free full heal and a free cooldown reset for the price of one finger — twenty seconds of
flight healed 180 health before that was fixed, and heals 1.6 now.

**Flight is a glide, not free ascent.** Once the air hops are spent, holding `Space` cuts gravity
to a sixth and floors the sink rate at 1.7 m/s, with enough air control to steer. You gain height
with the three jumps and reposition with apparition; twenty seconds of held throttle tops out at
7.5m and cannot leave the yard, so no second altitude clamp was needed.

**The mood is uniforms only, and it was tuned against a histogram.** No geometry moved. The
light rig comes down, the haze thickens, and the composite shader gains its own tone curve.

Getting that curve right took three attempts and the first two are worth recording. Attempt one
multiplied the whole frame down, which is the wrong operation: an overcast sky is emissive and
already sits near the top of the range, so a flat multiply took the yard — the darker half of
the image — to black while the sky kept its exposure. The result read as a skyline over a pit.
Attempt two overcorrected into something brighter than the operator's daylight, which was
readable and had no atmosphere at all.

What settled it was measurement rather than taste: render twelve fixed stances through the real
composite pass, histogram the bottom 62% of each frame — the part you have to walk through —
separately from the sky, and search the parameter space for the *darkest* setting that keeps the
walkable frame legible. The curve that won compresses highlights hard and shadows barely, which
pulls the sky down toward the ground, then lifts the black point only where the image is already
dark, so a fully shadowed interior does not turn milky.

Measured against the operator's daylight: mean ground luminance 0.20 against 0.26, so the yard
is a fifth darker; but 2.5% of the walkable frame falls below legibility against the operator's
18%, and on the worst stance 4.6% against 41%. The lesson is that the chosen light rig is
*dimmer* than the one that produced the unreadable frame. Darkness was never the problem — a
flat multiply and a contrast stretch that clipped everything below 5% to zero was.

**The incantation.** There is no voice actor and no audio files, so the whisper under every cast
is built the way the guns are: three sibilant noise bursts on a falling bandpass with the Q wound
up, staggered so it does not read as one flat hiss, over a low formant to put a throat behind it.
It is nowhere near words. The ear files it as a whisper anyway.

## The map

An 85m square — 7,225 square metres, twice the area of the 60x60 yard the game shipped with.
The original yard is untouched inside a 30m half-extent: the long container corridor, the
two-storey building with the ramp to the sniper deck, the open centre with the burnt-out
flatbed and the guard shack all sit exactly where they were. Everything from there out to the
fence is the outer apron, and it is authored rather than tiled — a second, longer container
corridor down the east side with two breaches cut into it, a container maze on the west so the
two flanks do not play the same way, a rank along the north fence, a loading dock with four
roller shutters and a ramp on the south, and watchtowers on the north corners whose decks sit
3.4m up, out of reach of a mantle from flat ground but reachable by a chained triple jump.

Prop density is held, not stretched. Against twice the area there are 2.02x the colliders,
2.24x the placed instances and 2.22x the cover anchors. The scatter passes that fill in litter,
tyre tracks and loose pipe now take their bounds and their counts from the map extent, because
left absolute they kept dropping the same debris inside the old boundary and left a bald ring
of swept concrete around it.

Every run of the outer container ranks is deliberately broken. A solid rank would seal the
outer lane off from the core, and a soldier whose patrol snapped into a sealed lane is one you
have to go hunting for at the end of a round. All four apron sides are verified reachable from
the player's spawn by the same flood fill the AI uses.

## Enemies

Soldiers patrol waypoints, routing around cover with a grid pathfinder over the same
colliders the player obeys. Spotting you takes a human 300 to 800ms before they react, and
they call it over the radio when they do.

Patrol routes are generated rather than authored, because the count is a setting and hand
placement does not scale to a hundred. Anchors are chosen by farthest-point sampling over the
floor a body can actually stand on, so the garrison fills the yard instead of bunching, and
each anchor gets a small loop whose radius shrinks as the yard fills. Ground anchors are
filtered through the same flood fill from the player's spawn that the waypoint snapper uses,
which is what stops a soldier being dropped into one of the pens that are open floor but
sealed by container stacks. A proportional share holds the upper deck at any count. Nothing
spawns within 17m of where you start.

As the squad thins, the radius of the "man down" callout grows, so survivors converge on you
rather than waiting to be found. That radius is proportional to the count, so the curve of a
ten-man round and a hundred-man round has the same shape.

In a fight they break for cover, strafe, flank, reload, and refuse to fire when their muzzle
is blocked even if they can see you over it. Line of sight is traced from the player's end
as well as theirs, which guarantees the two agree: if you cannot shoot them, they cannot
shoot you. Headshots do double damage, hits produce a flinch, and death drops them in a
ragdoll fall with the weapon coming loose.

## Everything is generated at runtime

There are no image, audio or model files anywhere in this repository.

Concrete, corrugated steel, plywood, rust, brick and chain link are painted into canvases at
load, down to the stencilled shipping marks on the container flanks. Weapons, hands and
soldiers are assembled from boxes, cylinders and spheres. The sky is evaluated per fragment
and the industrial skyline on the horizon is geometry.

Every sound is synthesised through the Web Audio API: per-weapon gunshots layered from a
crack, a body and a chest thump, reloads, footsteps, impact sounds that differ between wall
and flesh, the low-health heartbeat, and an ambient industrial hum with distant metal creaks
on a random timer. There is a convolution reverb built from a generated impulse response.

![Start screen](screenshots/menu.jpg)

## Performance

Dynamic resolution scaling watches frame times and adjusts render scale to hold the target
frame rate. Repeated props are drawn with `InstancedMesh` grouped by colour. Static scenery
is merged into single draw calls. Tracers, decals, shell casings and particles are pooled.
Shadow casting is trimmed by bounding-box size, which cut roughly two thirds of the shadow
pass for objects that only ever contributed a sub-pixel smudge.

Soldiers dominate the frame at high counts, because each one is 26 merged buffers and 8
shadow casters that are not shared between them. Doubling the map barely moved that: measured
draw calls per frame run about 280 at ten hostiles, 630 at twenty-five and 2,190 at a hundred,
against 594 at twenty-five on the old yard. Twice the props cost almost nothing in draw calls
because the new ones ride the existing instanced buckets. What the bigger map does cost is AI
time, since twice the colliders makes every `blocked` probe dearer — the worst case, the whole
garrison in combat at once, runs about 1.0ms, 2.2ms and 9.2ms against 0.5ms, 1.2ms and 4.6ms
before. The top of the slider is deliberately past what a mid-range machine will hold at 60fps;
the dynamic resolution scaler will not rescue it either, since the cost is draw-call submission
and CPU rather than fill.

Dark Lord mode is close to free on top of that. The whole spell layer — cooldowns, the wand
ember, the shield, the reveal shell, the silhouettes and the pooled cast lights — measures
0.6 to 0.7ms per frame with a hundred hostiles all revealed at once, and costs exactly one
extra draw call however many silhouettes are lit, because all of them are instances of one box
in a single `InstancedMesh` sized to three per soldier. The status effects are cheaper than the
AI they replace: a petrified or hoisted man returns from `updateEnemy` before perception,
pathing or tactics run at all.

Doubling the yard also doubles the per-cell work in two load-time passes. The navigation flood
fill goes from 14,641 cells to 29,241, which is 14ms. The patrol-anchor scan over the upper deck
needs a downward raycast per cell and would have been a 100ms-plus stall, so it samples at 2m
where the ground pass samples at 1m — it only has to supply a handful of anchors, and a quarter
of the rays is enough for that.

The sun's shadow ortho grew with the yard, from 104m to 148m across. Left at 2048 that would
have dropped shadow resolution from 20 texels per metre to 14, so the map went to 4096 and the
yard now gets 28 — sharper than the original rather than merely equal to it. The alternative,
keeping 2048 and tracking a tight ortho to the player, holds texel density for free but drops
shadows off distant geometry, which on a yard with 120m sightlines is the more visible loss.

## Implementation notes

Seven things are worth knowing before modifying `index.html`.

**`InstancedMesh.setColorAt` sizes its buffer from the wrong number in r128.** The
implementation is `this.instanceColor === null && (this.instanceColor = new
InstancedBufferAttribute(new Float32Array(3 * this.count), 3))` — `this.count` is the *draw*
count, not `instanceMatrix.count`. The Homenum Revelio silhouette mesh starts at `count = 0` so
that nothing draws until something is revealed, so the first `setColorAt` allocated a
zero-length array; every colour write after that silently ran past the end, and because
`instanceColor` was then non-null the shader still compiled with `USE_INSTANCING_COLOR` against
an empty attribute. The draw call was issued every frame and produced no pixels whatsoever.
Allocate `instanceColor` by hand at full capacity before anything touches it.

**No environment map.** three.js renders a metal with nothing to reflect as near-black, so
every material stays close to dielectric (`metalness` under about 0.2) and fakes metal
through colour and roughness instead. Raising `metalness` on the containers or weapons will
make any surface turned away from the sun collapse to black.

**Colour space is handled manually.** The renderer stays in linear output and the composite
shader does its own gamma encode, so material colours are pushed through
`convertSRGBToLinear()` at startup by `linearizeMats()`. New materials need the same
treatment or they will read oversaturated.

**Post-processing is hand-rolled.** The r128 CDN build has no `EffectComposer`, so bloom,
chromatic aberration, vignette, colour grading, damage tint and the low-health pulse all
live in one composite pass over a pair of render targets. Shadow tinting there is
multiplicative on purpose; an additive lift in linear space swamps near-black pixels and
turns the whole frame blue.

**The sky is a shader, and it has to be.** It began as a 4096x2048 equirectangular canvas,
which sounds like plenty until you do the arithmetic: 4096 texels over 360 degrees is 11
texels per degree, and the sniper's 15 degree field of view puts about 170 of them across a
1920 pixel screen. Eleven screen pixels per texel. Doubling the sheet buys one stop and costs
64MB, and it does nothing about the other half of the problem, which is that an equirect
projection collapses every column onto the zenith and smears whatever is painted up there
into a pinwheel. The replacement evaluates a gradient, a cloud deck and a cirrus layer per
fragment, so the scope magnifies ray directions instead of a bitmap. Clouds are projected
onto a flat slab overhead (`dir.xz / dir.y`), which bunches them towards the horizon the way
perspective actually does and makes the zenith an ordinary point of the plane rather than a
singularity. The fbm is band-limited against `fwidth` of that projection: octaves finer than
the fragment footprint are dropped rather than sampled, which is what stops the deck from
aliasing into a shimmer near the horizon where the slab stretches a degree of sky across
hundreds of noise cells. It measures free against a flat colour, because the dome is drawn
last of the opaque queue and early-z kills every sky fragment behind a wall.

**Sight alignment is data, not guesswork.** Each viewmodel returns an `adsPos` that puts its
sight on the viewmodel camera's -Z axis: `x` is zero and `y` is the negative of the sight's
height in model space. Move a sight and that number has to move with it. Optics you look
*through* need open-ended geometry with a double-sided material, since a capped cylinder
renders as a solid disc.

**The viewmodel is deliberately telephoto.** `VM_FOV` is 41.9 against the world's 75, and
every weapon sits proportionally further from the eye to hold its screen size. The two
numbers are a pair: narrowing the lens without pushing the gun out shrinks it, and pushing
it out without narrowing the lens makes it tiny. What the pairing buys is a flatter depth
gradient, so the buttstock stops ballooning over the receiver. `adsRef` is the sight's
distance from the eye and feeds the ADS dolly, so it scales with `adsPos.z`; `adsPos.x` and
`.y` do not, because sight alignment is independent of range.

Two smaller conventions: walkable surfaces are opt-out, since `box()` pushes anything solid
into `groundMesh` and `groundAt()` only considers surfaces at or below the probe height. And
the crosshair is drawn to a canvas rather than built from HTML elements, because at two
pixels thick every edge has to land on a whole device pixel or it smears.

## Repository layout

```
index.html              the entire game
PROMPTS.md              the prompts that specify it, verbatim
screenshots/            images used by this README
tools/test-harness.js   collision, weapon and AI test suite
tools/autoplay-bot.js   pathfinding bot for unattended regression runs
```

The two files in `tools/` are development-only. Neither is referenced by `index.html`; they
are pasted into the console or injected over the DevTools protocol. The harness exposes
collision audits, weapon accuracy and hit-fidelity tests, map boundary probes and an enemy
watchdog. The bot plays full rounds on its own with configurable aim error and reaction
delay, which is how the collision and AI fixes were regression tested.

## License

MIT. See [LICENSE](LICENSE).
