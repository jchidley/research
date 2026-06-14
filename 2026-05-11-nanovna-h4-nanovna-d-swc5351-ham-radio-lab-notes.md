# NanoVNA-H4, NanoVNA-D, SWC5351, and ham-radio lab use

Date: 2026-05-11

## Source session

- Session file: `/home/jack/.pi/agent/sessions/--home-jack-tmp--/2026-05-11T07-25-46-569Z_019e15ed-7ebb-749e-bb8b-558e16e947ad.jsonl`
- Session ID: `019e15ed-7ebb-749e-bb8b-558e16e947ad`
- Related earlier same-thread session containing RF Demo Kit / NanoVNA-F V2 troubleshooting: `/home/jack/.pi/agent/sessions/--home-jack-tmp--/2026-05-09T12-14-24-707Z_019e0ca9-07ba-7440-b44e-c4273c654b1e.jsonl`
- Local source inspected: `/tmp/NanoVNA-D`, cloned from `https://github.com/DiSlord/NanoVNA-D`; observed HEAD `9999d5e` and latest public release tag `v1.2.46` during this session.

## Working conclusion

For my intended use — amateur radio, Full Licence learning, QMX/QDX/QMX+ work, homebrew antenna systems, feedlines, chokes, transformers, matching links, filters, and test-board component experiments — the **NanoVNA-H4 running NanoVNA-D** is best understood as a practical, transparent, field-oriented radio instrument.

The H4 is strongest for HF/VHF work. Its extension above roughly 300 MHz is useful, but increasingly qualitative because the classic NanoVNA-H/H4 architecture uses odd harmonics of the clock source for upper frequency coverage. That is acceptable for insight, debugging, and relative comparisons, but not the same category as a lab VNA designed to do microwave measurements correctly.

The important differentiator is transparency: with **NanoVNA-D** I can inspect the actual firmware strategy for my hardware, including SWC5351 handling. This is not true in the same way for the NanoVNA-F V2 firmware I used, which behaved poorly with NanoVNA-Saver and is much less inspectable/adjustable.

## Hardware/firmware model

My H4 appears to be the newer **NanoVNA-H4 v4.4** family. Web research and DiSlord release notes indicate this should use:

- `Mode = SWC5351` in NanoVNA-D expert settings.

Practical action:

1. Set **CONFIG → EXPERT SETTINGS → Mode → SWC5351**.
2. Save/backup settings if available.
3. Recalibrate after changing this setting.
4. Treat high-frequency behaviour as invalid until calibration standards and known loads/thrus look sane.

## Code truth: NanoVNA-D supports multiple clock-source strategies

NanoVNA-D exposes three clock-generator modes in the UI:

```c
static const char* gen_names[] = {
  "Si5351",
  "MS5351",
  "SWC5351"
};
```

From `/tmp/NanoVNA-D/ui.c` around the expert mode selection code.

The selected mode chooses the band strategy table:

```c
void si5351_set_band_mode(uint16_t t) {
  static const band_strategy_t *bs[] = {
#if defined(NANOVNA_F303)
    band_strategy_H4_SI5351,  band_strategy_36H_MS5351, band_strategy_SWC5351
#else
    band_strategy_33H_SI5351, band_strategy_36H_MS5351, band_strategy_SWC5351
#endif
  };
  band_s = bs[t];
}
```

From `/tmp/NanoVNA-D/si5351.c`.

This means the setting is not cosmetic. It changes the frequency plan, divider plan, harmonic multipliers, output drive strengths, and gain compensation.

## Code truth: frequency limits and harmonic threshold

NanoVNA-D defines:

```c
#define FREQUENCY_MIN            600
#define FREQUENCY_MAX            2700000000U
#define FREQUENCY_THRESHOLD      300000100U
#define XTALFREQ                 26000000U
```

From `/tmp/NanoVNA-D/nanovna.h`.

Interpretation:

- Firmware allows a very low minimum frequency, down to 600 Hz.
- Firmware maximum is set to 2.7 GHz, but this is not a promise that the H4 hardware is quantitatively accurate there.
- The important threshold is about **300 MHz**. Above this, harmonic-mode behaviour becomes central.

## Code truth: SWC5351 band strategy

NanoVNA-D contains a specific SWC5351 table:

```c
// Mode for board H v3.6+ or H4 v4.3+ and SWC5351 installed
CONST_BAND band_strategy_t band_strategy_SWC5351[] = {
  {           0U,                0, { 0}, 0, 0,                            -1,                            -1, -1, -1,       1}, // 0
  {       32000U, SI5351_FIXED_PLL, { 6}, 1, 1, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_4MA,  0,  0,       1}, // 1
  {   130000000U, SI5351_FIXED_PLL, {40}, 1, 1, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_4MA,  0,  0,       1}, // 2
  {            1, SI5351_FIXED_MULT,{ 4}, 1, 1, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_4MA,  0,  0,       1}, // 3
  {   588000000U, SI5351_FIXED_MULT,{ 6}, 3, 5, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_4MA,  0,  0,   3*5*4}, // 5
  {            3, SI5351_FIXED_MULT,{ 4}, 3, 5, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_4MA,  0,  0,   3*5*4}, // 4
  {            5, SI5351_FIXED_MULT,{ 4}, 5, 7, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_8MA,  0,  0,   5*7*4}, // 6
  {            7, SI5351_FIXED_MULT,{ 4}, 7, 9, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_8MA, 40, 40,   7*9*4}, // 7
  {            9, SI5351_FIXED_MULT,{ 4}, 9,11, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_8MA, 40, 40,  9*11*4}, // 8
  {           11, SI5351_FIXED_MULT,{ 4},11,13, SI5351_CLK_DRIVE_STRENGTH_8MA, SI5351_CLK_DRIVE_STRENGTH_8MA, 40, 40, 11*12*4}  // 9
};
```

Interpretation:

- Below the first bands, the code uses fixed PLL modes.
- Direct/fundamental-style operation covers the useful low/mid range.
- Above the threshold, the table selects odd harmonic multipliers: 3, 5, 7, 9, 11.
- The table also changes drive strength and receiver gain.
- This confirms the H4 high-frequency extension is harmonic-based, not a clean microwave synthesizer architecture.

## How frequency generation works in the code

The key runtime path selects a band, then either uses fixed PLL or fixed multiplier mode:

```c
if (freq < band_s[1].freq) {
  rdiv = 7;
  drive_strength = SI5351_CLK_DRIVE_STRENGTH_2MA;
  band = 1;
} else if (freq <= 1000000U) {
  rdiv = 4;
  band = 2;
}
else
  band = si5351_get_harmonic_lvl(freq);
```

Then for fixed-multiplier operation:

```c
case SI5351_FIXED_MULT:
  fdiv = band_s[band].div;
  si5351_setupPLL_freq(SI5351_REG_PLL_A, (uint64_t)ofreq * fdiv, omul);
  si5351_setupPLL_freq(SI5351_REG_PLL_B, (uint64_t) freq * fdiv,  mul);
  ...
```

So the firmware does not simply ask the chip to generate the displayed RF frequency. It uses a band table and harmonic multiplier strategy.

## Practical frequency-confidence map

For H4 + NanoVNA-D + SWC5351:

| Range | Practical interpretation |
|---|---|
| LF/HF to 30 MHz | Excellent fit for QMX/QDX/QMX+ and general amateur radio work. |
| 50 MHz / 6 m | Well within the strong range. |
| 70 MHz / 4 m | Well within the strong range. |
| 144 MHz / 2 m | Still below 300 MHz; good confidence. |
| 430 MHz / 70 cm | Useful, but already harmonic-extension territory; calibrate carefully. |
| 900 MHz–1.5 GHz | Mainly qualitative unless verified against known standards. |
| >1.5 GHz | Experimental on H4 even if firmware permits display/sweep. |

This is why the H4 is a field radio instrument first, and not a substitute for a lab VNA at microwave frequencies.

## Relevance to amateur-radio antenna-system building

For antenna systems I care about:

- antennas
- transmission lines
- chokes
- transformers
- matching links
- baluns/ununs
- filters/traps
- patch leads and connectors
- QRP radio interfaces

The H4 + NanoVNA-D is the main workhorse because it directly measures:

- S11 / SWR / return loss
- complex impedance R+jX
- resonance and bandwidth
- feedline length and velocity factor
- cable faults/discontinuities via time-domain style transforms
- S21 insertion loss for filters, attenuators, transformers, and fixture experiments

NanoVNA-D explicitly contains measure modules aligned with this use:

```c
MEASURE_LC_MATH
MEASURE_SHUNT_LC
MEASURE_SERIES_LC
MEASURE_SERIES_XTAL
MEASURE_FILTER
MEASURE_S11_CABLE
MEASURE_S11_RESONANCE
```

This supports the interpretation that NanoVNA-D is designed around practical radio measurements, not just generic plotting.

## RF Demo Kit / test-board workflow

The RF Demo Kit guide found during the session says to calibrate using the board’s own fixtures:

- 13: Short
- 14: Open
- 15: Load
- 16: Thru

Then use the documented ranges for each test circuit. The most relevant examples:

- Circuit 5: 6.5 MHz ceramic trap/BSF, S21, 5.5–7.5 MHz
- Circuit 6: 10.7 MHz ceramic filter/BPF, S21, 9.7–11.7 MHz
- Circuit 10: inductor, S11 Smith, 50 kHz–30 MHz
- Circuit 11: 400 MHz LPF, S21, 100–600 MHz
- Circuit 12: 500 MHz HPF, S21, 1–1000 MHz

For component work on boards:

- Calibrate at the same reference plane as the DUT, ideally using the board’s own cal structures if that is what the DUT path uses.
- Use narrow spans around the behaviour of interest rather than huge wide spans.
- Recalibrate after changing span/point count where needed.
- Treat very small component values cautiously: fixture parasitics dominate small nH/pF measurements.
- Use the middle of a sensible frequency range, not the first couple of sweep points or frequencies where the reactance is too small to resolve well.

Observed examples from the session:

- A 430 pF capacitor behaving capacitive at 1–10 MHz and inductive above its self-resonance is normal.
- A small 40 nH inductor with about 10 ohm resistance is hard to measure at low MHz because reactance is tiny; around 100 MHz it becomes easier, and around hundreds of MHz self-resonance/parasitic capacitance dominates.
- These test boards are good for learning and trends, not precision de-embedded component metrology.

## tinySA Ultra: complementary role

The **tinySA Ultra** is not a replacement for the NanoVNA for antenna/feedline/choke/transformer impedance work.

Use the NanoVNA-H4 for:

- antenna match
- SWR/return loss
- feedline length/loss/faults
- choke impedance
- transformer/matching network response
- filter S11/S21

Use the tinySA Ultra for spectrum/signal work:

- transmitter harmonics and spurs
- QMX/QDX output cleanliness
- onboard/house RFI hunting
- checking oscillator leakage
- approximate signal levels
- signal injection for receiver checks
- before/after filtering spectrum comparison

Together, **NanoVNA-H4 + tinySA Ultra + oscilloscope** is a broader ham-radio bench than a NanoVNA-F V2 alone.

## NanoVNA-F V2 lesson

The F V2’s theoretical advantage is higher-frequency VNA coverage, roughly 1.5–3 GHz, using a V2-style architecture with ADF4350/AD8342-type parts rather than the H4 harmonic-extension approach.

But in this session, the F V2 was less useful practically:

- NanoVNA-Saver calibration failed: it read open and short as the same even though the device display itself showed them correctly.
- First sweep points appeared unreliable even with a calibrated load.
- Data above about 1.5 GHz looked suspect to the user.
- Firmware behaviour was less inspectable and less adjustable.

This does not prove all F V2 units are bad; it means for my workflow the H4 + NanoVNA-D is more trustworthy because it is stable, open, and understandable.

## LibreVNA comparison

The refined model:

- **NanoVNA-H4 + NanoVNA-D**: transparent, field-oriented, radio-practical instrument; excellent for ham radio and learning; high-frequency extension gives insight but not lab confidence.
- **LibreVNA**: lab/bench-oriented open instrument; intended for more correct higher-frequency VNA measurements.
- **NanoVNA-F V2**: potentially better GHz architecture than H4, but less transparent and in this session less cooperative with software.

The key is not merely frequency range. It is confidence in the measurement and ability to inspect the instrument’s method.

## How I will use this with AI/LLMs

Because NanoVNA-D source is inspectable, I can use LLMs as part of the lab process:

1. Ask the LLM to inspect firmware code paths for the exact hardware mode.
2. Identify band boundaries and harmonic transitions before interpreting unexpected trace features.
3. Compare weird measurement frequencies against firmware thresholds and band strategy tables.
4. Keep lab notes with:
   - firmware version
   - hardware mode
   - span/points
   - calibration reference plane
   - fixture used
   - observed artefact frequency
5. Treat code as truth where it describes device behaviour, and use web/spec sheets only to contextualise.

Potential future firmware experiments, if justified:

- adjust displayed/allowed frequency limits to discourage over-trusting high harmonic bands;
- add warning markers near harmonic band transitions;
- tune SWC5351 band table/gain settings only if I have external measurement evidence;
- add ham-radio-focused presets for antenna/feedline/filter work.

## Practical operating rules

1. Set H4 v4.4 to **SWC5351** mode.
2. Use H4 as the primary VNA for HF/VHF antenna-system work.
3. Trust measurements most below 300 MHz.
4. Treat 430 MHz as useful but calibration-sensitive.
5. Treat 900 MHz–1.5 GHz as qualitative unless verified.
6. Use tinySA Ultra for spectrum, spurs, harmonics, RFI, and signal injection.
7. Calibrate at the DUT/reference plane, especially on demo/test boards.
8. Do not overinterpret tiny nH/pF component readings without considering fixture parasitics and self-resonance.
9. Prefer the H4 + NanoVNA-D for learning because the implementation is open and inspectable.

## Useful learning resource

- YouTube playlist for using this class of tool: https://www.youtube.com/playlist?list=PL9Ox3wpnB0koBGofotI4xS8R0ct0FeYfv

Playlist scan on 2026-05-11 found a 13-video MegawattKS sequence focused on NanoVNA/tinySA use for radio design. Most relevant for my immediate ham-radio component/antenna-system work:

1. **NanoVNA Calibration - When, Why, and How to cal a VNA** — calibration concepts and workflow; should be watched before trusting any test-board/component result.
2. **NanoVNA Demonstrations - Coax line reflections and Smith charts** — directly relevant to feedlines, reflections, and Smith chart interpretation.
3. **NanoVNA - Measuring RLC Components** — directly relevant to demo/test board work and understanding parasitics/self-resonance in parts I will wire together.
4. **NanoVNA - Measuring Impedances** — useful for understanding measurement fixtures, terminations, instrument input impedance, and why setup matters.
5. **NanoVNA - Antennas and Tuners** — directly relevant to antenna system building and tuner/matching-network understanding.
6. **NanoVNA - Overview and antenna measurements with S11** — short practical introduction to antenna S11 measurements.
7. **NanoVNA and TinySA for Radio Design** — useful for the combined NanoVNA + tinySA bench workflow.

Other useful items in the playlist:

- **NanoVNA as a synthesized CW signal generator** — relevant for receiver testing, with external attenuation.
- **NanoVNA - Measuring S21 and S11 of a small-signal amplifier** — useful if measuring active stages; note need for attenuators.
- **NanoVNA - What is S11 and how good is the MFJ-264 dummy load?** and follow-up — useful for dummy loads, line/load impedance, SWR, and improving measurement setups.
- **Outfitting Microwave Labs on a Budget** — broader lab-equipment context, less central to HF antenna work but useful for future bench planning.
