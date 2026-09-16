# CapSoccer — bottle cap flicking game

A 3D physics game recreating the schoolyard bottle-cap game: a rectangular
table with a wide paper goal at each end, caps scattered across the board, and
players flicking their own caps — which physically collide with everything else
on the table — trying to land one in the far goal while defending their own.

Inspired by the "pen fight" recreation by
[@jaykosai](https://x.com/jaykosai/status/2098826870808519166), corrected to
match the real bottle-cap version: two wide goals (not a narrow gate), and a
full board of caps rather than one lane.

## Status

- [x] Full table with wide paper goals at both ends (matches the sketch)
- [x] Mode select screen — pick **Vs Computer (AI)** or **Two players (hot-seat)**
      before each match
- [x] Formation phase — each side places its 6 caps anywhere on the table
      (blue first, then red in two-player)
- [x] Alternate turns — blue flicks, board settles, red flicks, and so on
- [x] VS AI mode — the Red AI picks one of its caps, aims near the centre of
      your goal with imprecision, and sets power from distance to the goal;
      it only ever shoots toward your goal, never its own
- [x] Two-player hot-seat — both players place their own formation and take
      turns flicking their own caps on the same device, with a clear
      blue/red turn indicator in the HUD
- [x] Drag-back-and-release flick (slingshot style)
- [x] Cap-on-cap collisions (elastic, caps knock each other around)
- [x] Wall bounce on all four edges, friction, settle-to-stop detection
- [x] Goal / own-goal detection for **both** colours, per-colour scoreboard,
      flick counter
- [x] Game over — first side to run out of caps loses (goals decide if both
      run out together), or after `MAX_TURNS` flicks the score decides; a
      result screen offers a new match back to the mode select
- [x] Sound — all effects are synthesised (WebAudio, no audio files): flick
      whoosh scaled by power, cap-on-cap clank, wall/net thuds,
      placement click, goal jingle, own-goal sting, end-of-match fanfare, and
      a mute toggle in the corner
- [x] Playground dressing — dusk gradient sky, hanging lamps with warm light
      pools on a plank floor, backdrop wall, and proper goal frames (posts,
      crossbar, netted side walls, paper backboard); the side nets physically
      stop caps sliding out of the wide goal mouth
- [x] Punchy flicking feel — non-linear power curve (short pulls stay
      gentle, full pull blasts the cap ~half the table) with two-phase
      friction so fast caps glide then stop quickly instead of crawling
- [ ] Not yet built: finer balance tuning (see "Next steps")

## How to play

1. **Pick a mode** — play the computer (you're blue) or pass-and-play on one
   device.
2. **Formation**: click empty spots to place 6 caps wherever you want
   (blue places first; in two-player, red places next). In VS AI the red caps
   scatter automatically once you finish.
3. **Flick (on your turn)**: click one of your caps and drag backward — the
   aim line shows direction and power (turns orange near max power). Release
   to launch. Caps slide with friction, bounce off the table edges, and
   **collide with every other cap on the board** — you can deflect off the
   opponent's caps, get blocked by one, or send one flying.
4. A cap that comes **to rest inside the far goal** scores; landing in your
   own (near) goal removes it as an own goal. Scored and lost caps leave play
   for good.
5. **Restart** returns you to the mode select for a fresh match.

## Files

- `index.html` — the whole game (HTML + CSS + JS in one file, Three.js loaded from cdnjs)

## Physics notes (tunable constants at the top of the `<script>` block)

| Constant | What it controls |
|---|---|
| `FAST_FRICTION` (0.981) | Per-frame decay while a cap is moving fast (> 2 u/s) — the long powerful glide |
| `SLOW_FRICTION` (0.958) | Decay below 2 u/s — rolling resistance so caps stop snappily instead of crawling |
| `WALL_RESTITUTION` (0.55) | Bounciness off the table edges |
| `CAP_RESTITUTION` (0.85) | Bounciness of cap-on-cap collisions (elastic, equal mass) |
| `MAX_DRAG` (2.8) | How far you can pull back (world units) before the aim maxes out |
| `POWER_SCALE` (9.2) | Launch speed at 100% pull — reaches ~half the table |
| `POWER_CURVE` (1.6) | Exponent on the power meter: pulls stay gentle below ~60% and explode at the top end |
| `SETTLE_SPEED` (0.045) | Speed below which a cap is "stopped" — triggers the goal check once *all* caps are below this |
| `GOAL_W` / `GOAL_DEPTH` | Width and depth of each scoring zone |
| `BLUE_COUNT` / `RED_COUNT` (6 each) | How many caps each side has |

Game-flow constants (also at the top of the script):

| Constant | What it controls |
|---|---|
| `MAX_TURNS` (20) | Total flicks across both sides before the match ends on score |
| `AI_AIM_DELAY` (900) | Pause between your turn settling and the AI's shot |

Goal check only runs once every cap on the board has settled (not just the
one you flicked), since a flick can set off a chain of collisions.

### AI behaviour (VS mode)

The Red AI is deliberately simple — no pathfinding or lookahead:

1. It rates its caps by RHS column alignment + how close they are to your
   (bottom) goal, adds a little randomness, and picks the best one (among the
   top 3, with a 25% chance of just taking the best).
2. Its target is the centre of your goal, offset up to ±0.8 world units, with
   a small angle jitter (~0.14 rad).
3. Launch speed scales with the distance to the goal, clamped to a sensible
   range, with power jitter.
4. It never shoots toward its own (top) goal — the launch direction is always
   toward your goal, and a guard clamps the velocity so it can't drift
   backward into its own zone. Since red caps only ever rest outside goal
   zones, a backward shot would require it to cross the whole table.

## Next steps, in order

1. **Balance & rules polish** — formation-placement zone restriction (each
   side only places inside its own half), adjustable AI strength, and
   first-turn fairness for two-player (e.g. alternate who shoots first, or a
   lighter opening stock).
2. **Cap spin affecting bounce direction** — currently caps translate rigidly
   and spin is purely visual; adding tangential restitution would make
   deflections feel more physical.
3. **Real-time or remote multiplayer** — only once hot-seat feels right; would
   need a sync layer (WebSocket/WebRTC) since this is a self-contained client.