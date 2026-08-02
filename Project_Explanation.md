# Our Final Year Project — Explained Properly

Hey,

You asked what my project actually is, and I kept giving you a one-line answer that didn't really explain anything. So here's the full thing, written properly. No jargon where I can avoid it, and where I can't avoid it I'll explain the word first.

It's long, but you can read it in order and it should build up.

---

## Part 1 — The one-line version

We're taking over a project from a senior. He built a system that figures out **which direction a radio signal is coming from**. It works, but it has errors of about 15–20 degrees, and here's the thing:

**Nobody knows why.**

There are about six or seven different things that could be causing those errors, and they were all happening at once, so you can't tell which one is to blame.

Our job is to build a proper setup where we can test each cause **one at a time**, and finally answer: what is actually limiting this thing?

That's it. That's the project.

---

## Part 2 — What is "direction finding" anyway?

Imagine you're blindfolded in a field and someone is playing music from a speaker somewhere. You want to point at the speaker.

You'd probably cup your hand around one ear and slowly turn your head. When the music sounds loudest, that's roughly the direction. That's direction finding.

Radio direction finding is exactly this, but with an antenna instead of an ear.

### Why this is useful in real life

- Finding an illegal radio transmitter that's jamming a hospital's equipment
- Search and rescue — finding an avalanche beacon or an emergency locator
- Wildlife tracking — animals wear radio collars, researchers track them
- Telecom companies finding the source of interference in their network
- Military and police work, which is the angle my department cares about

Big systems that do this properly use **many antennas** arranged in a circle or a line, and they compare the tiny timing differences between them. Very accurate. Also very expensive, heavy, and fragile.

Our approach uses **one antenna that spins around**. Way cheaper. Way less accurate. The question is whether we can understand *why* it's less accurate.

---

## Part 3 — How the one-antenna method actually works

This is the core idea. Once you get this, everything else follows.

### Step 1: The antenna hears more in one direction

A directional antenna is like a cone-shaped ear. Point it at something and it hears that thing loudly. Point it away and it hears that thing quietly.

### Step 2: Spin it and record

You rotate the antenna in a full circle, and at every angle you write down how strong the signal is.

Say every 20 degrees: at 0° you record some number, at 20° another number, at 40° another, all the way around to 340°. That's 18 measurements.

Now you have a **curve**: signal strength versus angle.

### Step 3: Here's the part people get wrong

You'd think: just find the highest point on the curve, that's the direction. Done.

But that's not what's happening, and this is the whole thing.

**The curve is a blurred version of the truth.**

Think of a camera with a lens that's out of focus. You photograph a single tiny point of light. You don't get a point in the photo — you get a fuzzy smear. And critically, **the shape of that smear is a property of the lens, not of the light.**

Same lens, same smear shape, every single time.

In our system:
- The **out-of-focus lens** = the antenna's beam shape (called the "antenna pattern")
- The **smear** = the curve we recorded
- The **actual point of light** = the true direction of the transmitter

### Step 4: Un-blurring

Since the blur is caused by the antenna's beam shape, and the beam shape doesn't change, you can mathematically **reverse** the blur. Sharpen the photo back into a point. Then read off the angle.

All the fancy algorithm names you might have seen in my documents — **annihilating filter**, **FISTA**, **compressive sensing**, **beamforming** — are just different mathematical methods for doing this un-blurring. That's all they are. Different sharpening techniques, same job.

### Step 5: And here's the trap

Un-blurring only works if you know the lens shape **exactly**.

If your assumed lens is even a bit wrong, the sharpening doesn't just come out blurry. It comes out with **fake details** — extra bright spots that look exactly like real signals but aren't. They're artifacts of the mismatch.

This is a real thing in photography too. Over-sharpen a photo with the wrong assumptions and you get halos and ringing that were never in the scene.

**Remember this part. It's the centre of the whole project.**

---

## Part 4 — What my senior built

He built a working proof-of-concept, and honestly for what it was meant to be, it did its job. It showed the whole thing could be done with cheap equipment.

**His setup:**

| What | Details |
|---|---|
| Radios | Two ADALM-Pluto SDRs (~₹25,000 each). One transmits, one receives. |
| Antenna | A directional antenna on the receiver |
| Rotation | **By hand.** He physically turned it. |
| Steps | 20 degrees at a time — 18 measurements per circle |
| Location | Indoors, transmitter and receiver about **1 metre apart** |
| Processing | MATLAB, offline afterwards |
| Algorithms | Three of them, compared against each other |

**Quick note on "SDR":** Software Defined Radio. Normal radios have their tuning and filtering built into physical circuits. An SDR does almost everything in software — you tell it in code what frequency to listen to, how much to amplify, etc. It's a radio you program. That's why they're popular in research: one box, infinite configurations.

### His results

The true direction was **310 degrees**. His three algorithms said:

| Algorithm | What it reported |
|---|---|
| Annihilating Filter | 293.6° |
| Beamforming | 329.0° |
| FISTA | 322.0° |

So errors of about 16, 19 and 12 degrees. And the three algorithms disagreed with each other by 35 degrees.

He also got **extra peaks** — the algorithms reported additional directions where there was definitely no transmitter. He wrote that these were probably reflections off the room's walls and a metal chair.

That's a reasonable guess. But remember Part 3, Step 5? **A wrong lens shape produces fake peaks too.** And he never measured his lens.

---

## Part 5 — Everything that could be causing the error

This is the actual problem. Not that the error exists — that it can't be explained.

Here are the suspects. All of them were present at the same time in his experiment.

### Suspect 1: The lens was never measured

He didn't measure his antenna's beam shape. He measured the antenna's **physical dimensions** with a caliper, typed them into MATLAB, and let MATLAB **calculate** what the beam shape should theoretically be.

The gap between "calculated beam shape" and "actual beam shape" could be tiny or could be huge. Nobody knows, because nobody measured.

And a real antenna in a real setup isn't just the antenna. It's the antenna plus the metal mast it's mounted on, plus the cable, plus the tripod, plus whoever is standing next to it. All of that changes the beam shape.

> **This is my main suspect.** If the lens is wrong, everything downstream is wrong, and no amount of better maths fixes it.

### Suspect 2: Maybe 18 measurements wasn't enough

There's a rule in signal processing about how finely you need to sample something before you can reconstruct it properly. (It's related to why CDs sample audio at 44,100 times per second — sample too slowly and you get garbage.)

The same rule applies to sampling in **angle**. The rule here depends on how "complicated" the antenna's beam shape is. Simple smooth beam → few samples needed. Complicated beam with lots of bumps → many samples needed.

He took 18. Was that enough?

**Nobody knows**, because knowing requires measuring the beam shape (Suspect 1), and that wasn't done.

If 18 wasn't enough, the maths was fundamentally invalid from the start and every result is meaningless. That's a big deal, and it's answerable in about a week once we have the measurement.

### Suspect 3: Hand rotation

He turned the antenna by hand. So when the data says "20 degrees," the real angle might have been 18°, or 24°.

The algorithm trusts those angle labels completely. Wrong labels in, wrong answer out.

### Suspect 4: The radio's automatic volume control

This one is sneaky and I like it a lot.

Radios usually have **AGC** — Automatic Gain Control. It's the same thing your phone does in a video call: when audio is quiet it turns the volume up, when it's loud it turns it down. It's trying to be helpful.

But our **entire measurement is "how loud is the signal at each angle."**

If AGC is on, the radio is quietly flattening out exactly the variation we're trying to measure. It's like trying to weigh things on a scale that secretly adjusts itself to always read 50 kg.

We don't know if AGC was on during his experiment. Checking this is literally the first thing we're doing — it's one line in his code. If it was on, all his data is compromised in a way nobody noticed.

### Suspect 5: The room

1 metre apart, indoors, with walls and furniture. At that distance, signals bouncing off the walls arrive almost as strongly as the direct signal. So the "reflections" weren't a small disturbance on top of a good measurement — they were roughly as big as the thing being measured.

### Suspect 6: The code doesn't match the theory it's based on

This one took me a while to spot, and it's a bit technical, but it matters.

His work is based on a well-known 2010 research paper. But the implementation differs from that paper in three specific ways:

**(a) Wrong quantity.** The paper works with signal **power**. His code works with **voltage**, which is roughly the square root of power. The paper's maths is only valid in power — the moment you take a square root, the equations stop adding up correctly when there's more than one signal (which there always is, because reflections).

**(b) A missing step.** The 2010 paper has a cleanup step called **Cadzow denoising**. The paper's own results show that this step is *the reason* their method gets close to theoretical best-possible accuracy. Without it, the same method performs **10 to 100 times worse**.

His code doesn't have this step.

So when he concluded that FISTA was his best algorithm — he was comparing a fully-working FISTA against a deliberately handicapped competitor. That's not a fair comparison, and re-running it fairly is one of our experiments.

**(c) A shortcut in the un-blurring.** The paper uses a careful mathematical method. His code uses a faster, cruder one that only gives the same answer under specific conditions that probably weren't met.

### Suspect 7: The answer was picked by hand

Each algorithm produced **three** candidate directions, not one. His paper reports the one closest to the true answer.

But that's only possible because he already knew the true answer. In a real deployment you don't. So there's currently no automatic way for the system to pick — meaning **it doesn't actually have a measurable accuracy at all.** It has a shortlist that happens to contain the right answer.

### The core problem, in one line

> Seven things could be causing the error. All seven were present at once. Not one of them was measured separately. So the error can't be explained, and no improvement can be evaluated.

Think about what this means practically: if I invent a new algorithm tomorrow and it's 2 degrees better, I have **no way to know** whether my algorithm is genuinely better or whether I just got lucky with the room that day.

Until the causes are separated, further work on this system is uninterpretable.

---

## Part 6 — What we're actually doing

We're building a setup where these causes can be **switched on and off individually**, and measuring how much each one matters.

Like a doctor who won't prescribe anything until they've run tests to find out what's actually wrong, instead of guessing and hoping.

### The one-sentence version, for when someone asks

> "We're measuring whether the error in cheap radio direction finding comes from the maths or from the measurement, because nobody has separated the two."

### The four things we're building

**1. A motorised rotator.**
A motor that turns the antenna precisely, with a sensor called an **encoder** that reports the *actual* angle rather than the commanded angle. That distinction matters more than it sounds — see below.

**2. Software that records everything.**
Not just the signal. Also: the true angle from the encoder, the exact amplifier setting, whether AGC was on or off, the temperature, the timestamp, which antenna, which calibration file, and a reading with the transmitter **switched off** so we know the background noise level. Everything, every time, automatically.

**3. A corrected version of the algorithms.**
Fix the three deviations from the theory. Add the missing denoising step. Add an automatic rule for picking which peak is the answer, decided **before** we look at any data so we can't fool ourselves.

We also keep the senior's original code **completely unchanged**, because we need it as our "bad" comparison.

**4. A quality-control checker.**
Six automatic tests that every recording must pass before it's allowed into the analysis. Things like: did the signal overload the receiver? Did the amplifier setting stay constant? And my favourite:

> **Spin the antenna clockwise, record. Spin it back anticlockwise, record. The two should match.**
>
> If they don't, something mechanical is wrong — the motor has slack, or the cable is dragging. Physics doesn't care which direction you spun. Any disagreement is our equipment's fault, not the universe's.

Simplest test in the whole project and probably the most useful.

---

## Part 7 — The experiments

### Experiment 1: Measure the actual antenna beam shape

**The question:** what does the lens really look like?

**How:** outdoors, transmitter at a known bearing about 20–30 metres away. Rotate our antenna in careful 2-degree steps all the way around. Record everything. Now we have the real beam shape, measured with the mast and cable and everything present.

**What we get:**
1. The real lens, to use in the algorithms instead of the calculated one.
2. **How different the real lens is from the calculated one.** A single number. If it's large, we've just proven that a large chunk of the senior's error came from this, and we've proven it in the first two months.
3. Whether 18 samples was enough (answers Suspect 2).

**Why this is first:** everything depends on it. Also, this alone is a publishable result.

We also want to try **two or three different antennas** here, because of something genuinely counterintuitive I found in the 2010 paper:

> A "better" antenna — higher gain, cleaner, narrower beam — is a **worse** lens for this method.
>
> The maths works better when the beam shape is *complicated*, with lots of bumps and lobes. A clean narrow beam is a simple shape, which carries less information.

So a ₹500 hand-made antenna might genuinely beat a ₹12,000 professional one here. If that's true, it's backwards from what any textbook would tell you to buy, and it's worth a paragraph in the paper.

### Experiment 2: How many measurements do we actually need?

**The question:** is 18 samples enough? 36? 72?

**How:** record once at 2-degree steps (180 measurements). Then **throw data away** on the computer — keep every 10th to simulate 18 samples, every 5th for 36, and so on. Run the algorithms on each version.

One afternoon of recording, four different answers, zero extra fieldwork. Best effort-to-result ratio in the project.

### Experiment 3: The main one

This is the actual thesis result. It's a **2×4 comparison** — we vary two things independently and see which matters more.

**Thing 1 — data quality, two settings:**

| Setting | What it means |
|---|---|
| **Bad** | AGC on, calculated lens, hand-rotation angles, no noise reference — deliberately recreating the senior's setup |
| **Good** | AGC off, measured lens, encoder angles, noise reference, quality-checked |

**Thing 2 — algorithm, four options:** the senior's original, the corrected version with denoising, a simpler alternative method, and FISTA done properly.

That's 8 combinations. We run at least 20 recordings for each, so ~160 measurements. Then we ask:

- How much does switching **bad data → good data** improve things?
- How much does switching **worst algorithm → best algorithm** improve things?

**Whichever number is bigger, that's our answer.**

My prediction: data quality wins by a lot. Something like data quality improving things by 8 degrees while algorithm choice improves things by 1.5.

But — and this matters — **we might be wrong**, and that would also be a real result. If the algorithm turns out to dominate, that's genuinely useful information for everyone working in this area, because it means algorithm research on cheap hardware is well-founded after all. Either outcome is publishable. That's what makes it a real experiment rather than a demonstration.

### Experiment 4: Is it consistent?

Two different questions that get confused all the time:

- **Repeatability:** same setup, same day, same person, 20 recordings in a row. How much do they vary?
- **Reproducibility:** different day, different person, equipment taken apart and set up again, different location. How much do they vary now?

The second number is always worse and it's the one that actually matters in practice. Almost nobody reports it. We will.

### Bonus experiment (if we have time): The metal sheet

This is my favourite and I really hope we get to it.

Remember: the senior's extra peaks might be **real reflections** or might be **fake peaks from a wrong lens**. Both look identical.

Here's how you tell them apart:

1. Record in an open field. Note where the extra peaks are.
2. Put a big metal sheet at a known position. Record again.
3. **Move the sheet somewhere else.** Record again.

- If a peak **moves when the sheet moves** → it's a real reflection.
- If a peak **stays put** → it's a fake peak from the wrong lens.

Costs one sheet of metal and one afternoon. And it settles a question that multiple published papers have simply *assumed* the answer to without testing. That's a small but genuine contribution.

---

## Part 8 — Where machine learning fits (and where it doesn't)

Everyone asks this, so let me be specific.

### First, how much data will we actually have?

About 300–600 recordings by the end. Each one gives us: one direction estimate, the true answer, and maybe 15 recorded numbers (signal strength, temperature, angle error, and so on).

So: a table of roughly **500 rows × 15 columns**.

This matters enormously. That is **small data**. Deep learning — neural networks — needs tens of thousands of examples. What we have is exactly the size where simpler methods like **random forests** work really well and neural networks would just memorise the training set and fail on anything new.

Anyone who suggests a neural network here hasn't looked at the row count.

### What we ARE doing with ML

**Finding out which quality measurements actually predict a bad result.**

We feed the model all 15 recorded numbers and the resulting error, and ask it: which of these actually predicts when things go wrong?

The output is a **ranked list**. Maybe angle error matters most. Maybe signal strength. Maybe temperature matters not at all and we've been logging it for nothing.

Why this is a good fit:
- Uses data we're collecting anyway — zero extra fieldwork
- Correct data size for the method
- About 30 lines of Python
- **It approaches our main question from a second, independent direction.** Experiment 3 gives a careful answer about two things we deliberately controlled. The ML gives a rough ranking across all 15 things we merely recorded. If both point the same way, that's much stronger than either alone.

**Second thing, if time allows: predicting confidence.**

Instead of predicting the error, predict *"can this particular reading be trusted?"*

This is more useful than it sounds. In real use, the operator's question is never "what's the bearing" — it's **"should I drive two hours in that direction?"**

A system that says *"bearing 47 degrees, but the forward/reverse check disagreed and the signal was weak, so treat this as ±20 degrees"* is far more useful than one that confidently says 47 and is wrong.

It's also the thing you need if you ever want to combine readings from multiple locations to pinpoint a transmitter — you need to know how much to trust each reading before you can combine them sensibly.

### What we're deliberately NOT doing

**Training a neural network to output the direction directly.**

This is the obvious idea and it's wrong here, for four reasons:

1. Trained on simulated data, it would learn our simulator's quirks, not physics. It wouldn't transfer to reality.
2. No error guarantee. We couldn't compare it to the theoretical best-possible accuracy, and that comparison is our strongest piece of evidence.
3. It fails **silently**. Normal algorithms visibly degrade when things go wrong. Neural networks confidently output nonsense.
4. **It destroys the entire point of our thesis.** Our whole contribution is figuring out *where* the error comes from. Adding a black box means the error can now hide somewhere we can't look.

Point 4 is the real one. We'd be undermining the only thing that makes the project worth publishing.

### One thing we have to plan for from day one

If our quality checks work well, almost all our recordings will be good ones. Which means the ML model has almost no bad examples to learn from. Useless.

**So we deliberately keep the failures.** We run some recordings with AGC on, with wrong angles, with weak signals. We label them, exclude them from the main analysis, and feed them to the model.

The "bad data" half of Experiment 3 does double duty here — it's exactly the failure data the model needs.

---

## Part 9 — What we're buying

Total: roughly **₹20,000–45,000**, assuming we inherit the two radios and the antenna from the senior.

### The rotator — most important purchase, ~₹10,000–18,000

Built from parts rather than bought as a unit:

- **NEMA 23 stepper motor** — a smaller one would lose steps under the antenna's weight and wind, and then our angle log becomes fiction
- **Planetary gearbox, 10:1** — more torque, finer steps. Planetary specifically because other gearbox types have too much slack, and slack is exactly the error we're trying to measure
- **AS5048A magnetic encoder** — reports the true angle to 0.02 degrees, which is so much better than what we need that encoder error just drops out of our analysis entirely
- Driver, microcontroller, power supply, bearing, mounting plate

**Two things that matter a lot mechanically:**

1. **The encoder goes on the output shaft, after the gearbox.** Before the gearbox it tells you where the *motor* went. After, it tells you where the *antenna* actually is, including all the slack. Get this backwards and we've built an expensive way to log the same commanded angles the senior had — and Experiment 3 has nothing to compare.

2. **The antenna's centre must sit exactly on the rotation axis.** If it's off-centre, rotating it changes its distance to the transmitter slightly, which changes the signal strength, which looks *exactly like* a real feature of the antenna's beam shape. That would poison Experiment 1, which everything else depends on.

### RF filter — ~₹2,000–8,000, non-negotiable

We work at 700–800 MHz, which is a **mobile phone band**. Nearby phone towers transmit there at enormous power compared to our little test transmitter.

Without a filter, those tower signals overload the receiver, and once it's overloaded, our "how strong is the signal" measurement — **which is our entire measurement** — becomes nonsense that still looks perfectly plausible.

Cheapest insurance in the project.

### Calibration kit — ~₹5,000–12,000

- **Step attenuator** — lets us weaken the signal by known exact amounts. So we can check the radio reads correctly, and sweep signal strength in experiments without physically moving anything
- **Splitter and cables** — connect transmitter to receiver directly, no antennas, to test the electronics on their own
- **Temperature sensor (₹100)** — a data point for the ML

### Antennas — ~₹300–5,000

Two or three, to compare. The senior's LPDA, plus a hand-built Yagi or a patch antenna etched on a circuit board. Because of the counterintuitive thing in Part 7 — the cheap ugly one might win.

### A ₹500 oscillator upgrade

The Pluto radios come with a cheap internal clock that drifts with temperature. Swapping in a better one is one afternoon of soldering and removes a whole category of annoying errors permanently.

### What we're NOT buying

- **A ₹45,000 second radio system** for cross-checking — great idea, no time in 6 months
- **A powerful computer** — our laptop is plenty. The maths takes under a tenth of a second; the motor takes several seconds. Speeding up the maths would be optimising the wrong thing entirely
- **GPS positioning (₹30,000+)** — at 30 metres range, a tape measure and basic trigonometry gives us better accuracy than we need

### Before spending anything

Walk through the ETC lab and see what already exists. Antenna labs often have signal generators, spectrum analysers, calibrated attenuators. Every one we can borrow is a line item deleted.

---

## Part 10 — The plan over 6 months

Two of us. One does hardware and recording software. One does the maths and analysis. Both go to every field experiment, because two people are needed and it halves the mistakes.

| Weeks | Hardware person | Software person |
|---|---|---|
| **1–4** | Build the rotator, test its accuracy | **Check if AGC was on in the senior's code** (this is genuinely the first task), reproduce his results, fix the three code problems |
| **5–8** | Recording software, metadata logging, calibrate the electronics | Finish all four algorithms, write the automatic peak-picking rule |
| **9–12** | *Both:* Experiment 1 — measure the antenna beam shape | |
| **13–16** | Quality control checks, build the "bad data" setup | Write all analysis code **in advance**, tested on fake data |
| **17–21** | *Both:* Experiments 2, 3 and 4. Metal sheet test if there's time | |
| **22–26** | *Both:* ML analysis, write the paper, package the data | |

### Where this could go wrong

Weeks 1–8 are the risk. If the rotator isn't producing reliable data by week 10, we cut Experiment 4 first, then Experiment 2, and protect Experiment 3 at all costs.

**Experiment 3 alone is a paper.** Everything else is bonus.

### The safety net

Experiment 1 gives us a real result by around week 10 — "the antenna beam shape the previous work assumed differs from reality by X." That's a concrete finding about published work, obtained in under half our time.

Even if everything after that goes badly, we have something.

---

## Part 11 — What we're honestly NOT doing

I want to be upfront about this, because scope creep is how these projects die.

**Not doing:**
- Locating a transmitter's actual position — we only find *direction* from one spot. Position needs two or more locations working together
- Real-time operation — we record now, process later
- Tracking anything that moves
- A nice user interface or map display
- Any claim that this is deployable in the field

**Not claiming:**
- A new algorithm — we're using existing ones, that's the point
- Competing with professional equipment — commercial systems get under 2 degrees instantly and always will
- That our system is accurate — we're measuring *why* it isn't

Saying all this upfront reads as knowing what you're doing. Getting caught overclaiming reads as the opposite.

---

## Part 12 — Why this is actually worth doing

### It's a real gap, not a made-up one

Every published paper in this area tests in ideal lab conditions — special echo-free rooms, expensive measurement equipment, properly measured antennas. They report one overall accuracy number.

None of them ask: **what happens when you build this out of cheap parts, and where does the error come from then?**

That's the question. It's specific, it's answerable in 6 months, and the answer is useful to everyone else in this area.

### It's honest about being infrastructure work

We're not inventing a new algorithm. We're building the measuring instrument that lets you find out whether new algorithms are worth inventing.

Nobody publishes a paper called "we built a weighing scale." They publish what they weighed.

### The result is actionable either way

If it turns out calibration dominates: everyone working on this should stop tuning algorithms and start measuring their antennas.

If it turns out the algorithm dominates: algorithm research on cheap hardware is justified, carry on.

Either way, the next person doesn't have to guess.

---

## Part 13 — If someone challenges you about it

**"So you built a spinning antenna. That's it?"**

Yes — and deliberately so. The rotator is the *instrument*, not the result. It had to be boring and predictable, because anything clever in the hardware would just have been another thing confusing the results. What we built it *for* was to measure which parts of a cheap direction-finding system actually limit accuracy, and nobody had measured that.

**"Why not just use better algorithms?"**

Because you can't tell if an algorithm is better until you know how much of your error is coming from somewhere else entirely. If I improve the maths by 2 degrees but the antenna calibration is costing me 10, I've optimised the wrong thing and I wouldn't even know.

**"Isn't this just engineering, not research?"**

The building is engineering. The result — a measurement of which error source dominates, with actual numbers — is research. It doesn't exist anywhere in the literature.

**"What if you find nothing?"**

Not possible. Every outcome is a result. If calibration dominates, that's a finding. If the algorithm dominates, that contradicts our expectation, which is a more interesting finding. The only failure mode is not collecting enough data to tell — which is exactly why the schedule protects Experiment 3 above everything else.

---

## Part 14 — The whole thing in ten lines

1. A spinning antenna can tell you which direction a radio signal is coming from
2. The recorded curve is a **blurred** version of the answer; the blur is the antenna's beam shape
3. Algorithms **un-blur** it — but only work if you know the beam shape exactly
4. Our senior got 15–20 degrees of error and some fake extra directions
5. Seven different things could be causing that, all present at once, none measured separately
6. So the error can't be explained, and no improvement can be evaluated
7. We're building a setup where the causes can be switched on and off individually
8. Main experiment: same algorithms on **carefully collected** vs **sloppily collected** data
9. Whichever change matters more — better data or better algorithm — is our answer
10. Nobody has measured this, and everyone in the field is currently guessing

---

That's the whole thing. Ask me about any part of it — especially if something in Part 3 didn't land, because everything else is built on that one idea about the out-of-focus lens.

Also, if you're free around week 9, I could genuinely use a third person holding a tape measure in a field.
