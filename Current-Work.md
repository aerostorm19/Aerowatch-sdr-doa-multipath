# What I've built so far — explained simply

This is for you to understand what my Direction Finding project code actually does,
in plain English, no jargon dump. I'll go step by step.

---

## 1. The overall idea

Imagine you have one antenna, and somewhere in the room (or field) there's a radio
transmitter. You don't know which direction it's transmitting from. If you rotate
your antenna slowly, all the way around, and at every angle you note down "how
strong is the signal right now," you get a curve — strong when you're pointing at
the transmitter, weak when you're not.

That curve is basically your raw data. Everything else is different ways of taking
that curve and answering: **"which angle was the transmitter actually at?"**

I'm testing 5 different algorithms that all try to answer that same question, in
different ways. Some are simple, some are smarter, some solve harder versions of
the problem (like "what if there's an echo/reflection messing up my reading?").

---

## 2. The first three algorithms — all working on the "rotate and measure" data

These three all take the same input: the strength-vs-angle curve I described above.
This part is based on a real paper from my own department (AIT Pune) that actually
built this with SDR radios and tested it indoors.

### A. Annihilating Filter (AF)

Think of it like this: my antenna doesn't pick up signal from *just* the direction
it's pointing at — it also picks up a bit from nearby angles (like a flashlight beam
that isn't a perfectly sharp line, it has a bit of spread). So my strength-vs-angle
curve is a "blurred" version of the truth.

AF basically **un-blurs** the curve mathematically, then does some matrix algebra
(nothing to worry about the details) that spits out the exact angle(s) where the
signal is coming from.

**What I found:** It works well in my simulation — it found the two directions I
planted almost exactly.

### B. Beamforming

Same un-blurring step as AF, but instead of solving equations to get an exact
answer, it just **scans through every possible angle** and checks "does this angle
match my data well?" The angle that matches best is the answer. Like literally
trying every possible answer and picking the winner.

**What I found:** Also worked well, gave basically the same answer as AF in
simulation — expected, because they both start from the same un-blurred data.

### C. FISTA (Compressive Sensing)

This one assumes something extra: that there are only a *few* transmitters, not
signal everywhere. So instead of un-blurring, it tries to explain my measured
curve using as few "spikes" (candidate directions) as possible. It has a knob
called **lambda** that controls how few spikes it's allowed to use — turn it up,
it becomes stricter about using few spikes; turn it down, it's more relaxed.

**What I found:** Worked in simulation too. But — and this is straight from the
original paper, not something I found — when they tested this outdoors/indoors
with real reflections, this lambda knob had to be tuned differently for real data
vs. clean simulated data. That's a known weak point, not something I'm hiding.

---

## 3. Why these three aren't enough — the multipath problem

Here's the catch: in a real room, my antenna doesn't just get signal directly from
the transmitter. The signal also bounces off walls, furniture, etc., and reaches my
antenna a tiny bit later, from a different apparent angle. So my "strength vs angle"
curve doesn't just have one bump at the true direction — it has extra fake bumps
from reflections.

The original paper actually ran into this: when they tested with a real transmitter,
all three algorithms above found some consistent "fake" peaks that weren't the real
transmitter — caused by the room's walls and a metal chair reflecting the signal.

This is the real problem I'm now trying to solve, and it's why I added two more
algorithms.

---

## 4. Algorithm 4 — CMA (Constant Modulus Algorithm)

This one works differently — it doesn't look at the strength-vs-angle curve at all.
It looks at the raw radio wave itself, over time.

Here's the trick: the type of signal we're using (QPSK, a common digital radio
modulation) always has the exact same *strength* at every instant — only its phase
changes, not its amplitude. Picture it like a clock hand that's always the same
length, just pointing in different directions over time.

When there's an echo/reflection, and it adds to the direct signal, the combined
wave's strength starts wobbling — the "clock hand" isn't constant length anymore,
because two waves are adding together and sometimes helping each other, sometimes
cancelling.

**CMA's job**: watch the wobbly signal, and slowly adjust a filter until the
signal's strength becomes constant again — basically undoing the mess the
echo/reflection caused. It doesn't need to be told what the echo was — it figures
it out purely by trying to make the output "flat" again.

**What I actually tested:** I made a fake signal — a direct copy plus one echo,
added some noise — and ran CMA on it.

- **Before CMA:** the signal's strength wobbled a lot (measured as 0.39 — this
  number is basically "how much the strength varies," 0 would mean perfectly
  constant).
- **After CMA:** the wobble dropped to 0.13 — about 3x cleaner.
- I also plotted the "constellation" (a scatter plot of the signal's I/Q values —
  think of it as a snapshot of all the "clock hand" positions). Before CMA, this
  scatter is smeared into a blob. After CMA, it should start clumping back into 4
  distinct clusters (since QPSK only ever sends 4 possible symbols).

**What this proves:** the "cleanup" step actually works on fake data.

**What this does NOT do yet:** it doesn't give me a direction/angle. It only
cleans the signal. Turning "cleaned signal" into "here are 2 separate directions,
one for the direct path and one for the echo" is the next stage (called
Separation + Estimation in the paper I'm following), which I haven't built yet.
So right now CMA is a tested building block, not a finished pipeline.

---

## 5. Algorithm 5 — MUSIC

MUSIC is a completely different, older, well-known technique from array signal
processing — but there's a catch, explained below.

**How MUSIC normally works:** you need *multiple antennas* at once (an array),
not just one rotating antenna. Each antenna gets a slightly different version of
the same signal depending on the direction it's coming from. MUSIC does some
matrix math (called eigen-decomposition, don't worry about it) on all these
readings together, and separates "this is real signal" from "this is just noise."
The directions where the "signal energy" is strongest are your answer.

**What I actually tested:** since I don't have multiple antennas yet, I built a
fake scenario — 8 imaginary antennas, 2 fake transmitters at 20° and -30°, and ran
MUSIC on it.

- **Result:** it found both angles exactly right — 20° and -30°.
- I also looked at something called eigenvalues (a list of 8 numbers, one per
  antenna) — the first 2 were big (9.18 and 6.79) and the rest were small and
  close together (all under 0.08). That big jump is exactly what tells MUSIC
  "there are 2 real signals here, the rest is just noise." Seeing that clean jump
  means my code is doing the math correctly.

**What this proves:** my MUSIC code is written correctly and works on the kind
of data it's designed for.

**What this does NOT prove:** that MUSIC works on my actual setup. My real
hardware only has ONE antenna, not 8. MUSIC needs multiple simultaneous readings
to build that matrix — a single rotating antenna doesn't naturally give you that.
So right now this is "I understand and can code MUSIC," not "MUSIC works for my
project." Making it work with one rotating antenna is still an open problem I
haven't solved — it would need some way of treating the different rotation angles
as if they were an array, which needs the readings to be phase-consistent with
each other (tricky with a hand-rotated antenna).

---

## 6. Honest summary — what's actually done vs. not

| Algorithm | Is it working? | On what kind of data? | Ready for my real hardware? |
|---|---|---|---|
| Annihilating Filter | Yes | Simulated rotate-and-measure curve | Yes |
| Beamforming | Yes | Simulated rotate-and-measure curve | Yes |
| FISTA | Yes | Simulated rotate-and-measure curve | Yes (but needs tuning per environment) |
| CMA | Only the "cleanup" part is done | Fake 2-path echo signal | No — missing steps to turn it into an angle |
| MUSIC | Code is correct | Fake 8-antenna array | No — my hardware doesn't have an array |

Nothing here is fake or exaggerated — it's genuinely working code, I've run it and
checked the numbers make sense. I'm just being precise about what each result
actually proves, so I don't end up overselling something that isn't finished yet.

**Next thing I'm going to build:** connect CMA's cleaned-up signal to an actual
angle estimate (the "Separation" step), so I get a real multipath-resistant DoA
result out of it — that's the actual missing link right now.
