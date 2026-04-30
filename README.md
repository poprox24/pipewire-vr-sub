# pipewire-vr-sub

A PipeWire filter-chain config that takes stereo audio from a VR application (WiVRN, SteamVR, etc.) and routes it to your subwoofer's LFE channel - with a fully tunable parametric EQ in between.

Tested on Linux with WiVRN, but any app you can route through PipeWire will work.

---

## What it does

```
VR App (stereo) → virtual sink → mixer (FL+FR → mono) → parametric EQ → LFE out
```

1. Creates a virtual stereo audio sink (`vr_sub_eq_in`) you can route apps into
2. Sums both channels to mono at unity gain
3. Runs the mono signal through a 5-band parametric EQ you can tune to your sub
4. Outputs the result to your audio device's LFE channel

---

## Requirements

- PipeWire (with `pipewire-pulse` if you use PulseAudio apps)
- A audio device with a dedicated LFE/subwoofer output channel

---

## Installation

1. Copy the config file to your PipeWire config directory:

```bash
mkdir -p ~/.config/pipewire/pipewire.conf.d
cp vr-sub-eq.conf ~/.config/pipewire/pipewire.conf.d/
```

2. Restart PipeWire:

```bash
systemctl --user restart pipewire pipewire-pulse
```

3. Wire up the input/output manually - the node won't auto-connect by design, because many apps create the sink on program startup(buggy with Pipewire). Use a patchbay tool like **qpwgraph** or the `pw-link` CLI to connect `vr_sub_eq_in` to Pipewire and `vr_sub_eq_out` to your subwoofer's LFE channel.  
  (Note, this needs to be ran every time you restart your computert, for programs such as WiVRn I recommend adding it as an autolaunch script with a sleep/delay of 3 seconds)


---

## Tuning the EQ

The config includes a starting point tuned for a specific setup - you'll almost certainly want to adjust it for yours.

Open the config file and find the `filters` block:

```
{ type = bq_peaking  freq = 40     gain = 0.95  q = 0.7  }  # Sub-bass body
{ type = bq_peaking  freq = 65     gain = 2.7   q = 1.2  }  # Upper sub-bass boost
{ type = bq_peaking  freq = 103.8  gain = 1.1   q = 1.15 }  # Low-mid presence
{ type = bq_peaking  freq = 350    gain = 0.67  q = 2.0  }  # Mud reduction
{ type = bq_peaking  freq = 700    gain = 1.5   q = 1.6  }  # Upper crossover region
```

Each band is a bell (peaking) filter. Parameters:

| Parameter | Description |
|-----------|-------------|
| `freq`    | Center frequency in Hz |
| `gain`    | Boost (positive) or cut (negative) in dB |
| `q`       | Bandwidth - higher Q = narrower, more surgical |

After saving changes, restart PipeWire again to apply them.

**Tip:** Tools like [EasyEffects](https://github.com/wwmm/easyeffects) can help you measure your sub's response and figure out what corrections it needs.

---

## Adjusting the mixer gain

If your sub sounds too quiet or is clipping, adjust the two gain values in the mixer node:

```
control = { "Gain 1" = 0.5 "Gain 2" = 0.5 }
```

Both channels are summed, so `0.5 + 0.5 = 1.0` is unity. Lower both values proportionally to reduce the level, e.g. `0.35 + 0.35`.

---

## Manual wiring with pw-link

If you prefer the CLI over qpwgraph:

```bash
# List available ports to find your app and LFE sink
pw-link --output
```
```bash
# Connect the input and EQ output to your subwoofer

pw-link wivrn.sink:monitor_FL vr_sub_eq_in:playback_FL
pw-link wivrn.sink:monitor_FR vr_sub_eq_in:playback_FR
pw-link vr_sub_eq_out:output_LFE <your-lfe-sink>:<port>
```
You may replace wivrn.sink with your chosen app.

---

## License

MIT - do whatever you want with it.
