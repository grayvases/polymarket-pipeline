# Metronome Sync

An interactive simulation of the classic demonstration where a crowd of metronomes, each set to its own tempo
and standing on one shared board that can roll, slowly fall into step.

Open `index.html` in a browser. It is a single file with no build step and no dependencies
(web fonts load from Google Fonts and fall back to system fonts when offline).

## What you can do

- Choose how many metronomes (2 to 40) and the tempo range (30 to 240 BPM). Each metronome gets a random tempo in that range.
- Watch the **sync percentage** climb, see how many are **locked**, the **group tempo** (with its Italian name),
  and the **live BPM of every metronome that has not synced yet**.
- Every metronome has its live BPM above it. The weight on its rod marks the tempo you set.
- Drag along a rod to slide the weight and retune that metronome. Click one to knock it out of step.
- Coupling strength is measured against the exact threshold at which all of your tempos can lock (marked "all lock").
  Auto-ramp raises it slowly so metronomes join the beat one at a time. Moving the slider yourself switches the ramp off.
- Speed: any whole number from 1× to 1000× (slider, number box or quick buttons). The physics gives the same result at every speed; at 1000× a 90 second ramp takes about a fifth of a second. Tick sounds switch off above 6× and the tick flashes above 40×.
- Sound: each metronome ticks at its own pitch (pentatonic, slowest is lowest), so you can hear a chord form as they line up.
- **Exact sync sound effect.** When every pendulum is within 3° of the crowd's average (about 9 ms at 100 BPM, tight enough that the
  ticks fuse into a single click), the sync reads 100% and a sound effect plays: a low impact, a bright chord that rings out, an upward
  sparkle and an air sweep. A soft two-note ping plays earlier, when everyone has locked but is still a few degrees apart.
  "Preview the exact-sync sound" in the Sound section plays it on demand. Knocking a metronome out of step and watching it snap
  back does not replay it; losing exact sync for more than a second and a half does.
- Charts: a phase ring, every metronome's live tempo over time, and sync over time.

Keyboard: `Space` pause, `R` restart, `N` new tempos, `M` mute, `C` controls, `?` help.

## How it works

Each metronome is a phase oscillator θᵢ. The board couples them all through a single shared signal:

```
dθᵢ/dt = ωᵢ + K · r · sin(Ψ − θᵢ),     r·e^{iΨ} = (1/N) Σ e^{iθⱼ}
```

A pendulum's angle is `0.42·sin θ` and it ticks at both extremes, so BPM = 60·ω/π. This is the Kuramoto model, which
Pantaleone (2002) showed is what metronomes on a movable base actually do. The platform recoils against the mean
pendulum swing, which is what you see it do (the cans underneath roll).

Details worth knowing:

- **Lock threshold.** For the exact tempos on screen, the weakest coupling at which a fully locked state exists is
  `K_L = min over u ≥ max|Δᵢ| of u / r(u)`, with `r(u) = mean √(1 − (Δᵢ/u)²)` and `Δᵢ = ωᵢ − mean(ω)`.
  The coupling slider is expressed as `K / K_L`.
- **Sync %.** The order parameter `r`, corrected for the alignment N random phases show by chance
  (`≈ 0.886/√N`), so that "0%" means unsynced even for small N. The display stops at 99% until sync is exact.
- **Exact sync.** Locked metronomes still sit a few degrees apart, because a finite coupling leaves each one a phase offset of
  `asin(Δᵢ / K·r)`. Exact means the widest offset from the crowd's mean phase is under 0.05 rad (2.9°) for 0.6 s with coupling at or above
  the lock threshold, and it is lost beyond 0.25 rad (14°).
- **Locked.** Live tempo is a smoothed `dθ/dt`. Locked means within a small window of the largest tempo cluster, and
  that cluster's phases are coherent beyond chance. A hysteresis stops the labels flickering.
- **Auto-ramp.** Coupling rises from 0.35 K_L to 1.05 K_L over 70% of the chosen time (where the cluster forms and
  recruits), to 3.6 K_L over the next 12% (everyone locks), then geometrically to `40 × max|Δᵢ|` over the last 18%. That final value
  shrinks the phase offsets to about 1.4°, which is what makes exact sync reachable. The coupling slider's last 10% of travel
  reaches the same territory by hand (up to 50 K_L).
- **Integration.** Heun's method in simulated time, independent of the display's frame rate: 240 Hz, with a smaller step when
  coupling gets very stiff at the end of the ramp.
- **Audio.** Web Audio, one short pitched wood-block voice per tick, scheduled on the audio clock and run through a compressor.

If the frame rate stays low, the app drops backdrop blur, glow haze and canvas resolution automatically.

## Sources

- C. Huygens, letters on two pendulum clocks sharing a beam, 1665.
- J. Pantaleone, *Synchronization of metronomes*, Am. J. Phys. 70, 992 (2002).
- Ulrichs, Mann and Parlitz, *Synchronization and chaotic dynamics of coupled mechanical metronomes*, Chaos 19 (2009).
- Boda, Néda, Tyukodi and Tunyagi, *The rhythm of coupled metronomes*, Eur. Phys. J. B (2013).
- Y. Kuramoto, *Chemical Oscillations, Waves, and Turbulence* (1984).
