# Customized Parts

This page collects alternatives to the parts in the [main parts list](README.md) and the [extra parts](extra_parts.md), plus the workspace items we chose for the build. They are **not part of the official build** and have not been validated by the maintainers. Each rover entry explains what changes, why, and what you need to adjust elsewhere on the rover. Use them at your own risk.

**Rover**

1. [Ryobi ONE+ 18V battery power supply](#1-ryobi-one-18v-battery-power-supply)

**Workspace**

2. [Workbench mat: BSONEDO silicone mat, 24" x 80"](#2-workbench-mat-bsonedo-silicone-mat-24-x-80)
3. [Floor: MotoFloor Modular Garage Flooring Tiles](#3-floor-motofloor-modular-garage-flooring-tiles)
4. [Solder fume extractor: KOTTO Solder Smoke Absorber](#4-solder-fume-extractor-kotto-solder-smoke-absorber)
5. [Bench DC power supply: SPS-3010, 0–30V / 0–10A](#5-bench-dc-power-supply-sps-3010)

## 1. Ryobi ONE+ 18V battery power supply

If you already own Ryobi ONE+ 18V tool batteries and chargers, you can use them to power the rover instead of the recommended 4S LiPo. This saves the cost of a LiPo, a balance charger and a fire-safe bag. It also gets you hard-cased, self-protecting packs that can be swapped in seconds and charged on a charger you can't misconfigure.

The catch is voltage. A ONE+ pack is a 5-cell (5S) Li-ion battery: about 20.5–21V when fully charged, 18V nominal, and about 15V when empty. A 4S LiPo runs from 16.8V down to about 14V. The electronics handle the higher voltage, but you must change some Roboclaw and software settings. You also have to add your own low-voltage cutoff.

### Compatibility at a glance

| Component | Rating | At 21V |
| --- | --- | --- |
| 5V regulator (Pololu D24V150F5) | 40V max input | OK |
| 12V regulator (Pololu D24V22F12) | 12.7–36V input | OK |
| Roboclaw 2x7A | 34V max | OK, but the **Max Main Battery setting must be raised** (see below) |
| INA260 | 36V max | OK |
| Electrolytic capacitors | 50V | OK |
| Main fuse | 10A, 125V | OK |
| D1 (STPS10L25D Schottky) | 25V | OK, but only ~4V of headroom. Don't go above 5S. |
| DC power meter (DROK, Amazon B017FSED9I) | 8–100V, 20A | OK |
| Steering servos, encoders | Powered from the 5V rail | Not affected |
| Drive motors (goBILDA 5203, 12V nominal) | — | Overvolted at high duty. Mitigated by the software change below. |

### What you need

| Part | Qty | Notes |
| --- | --- | --- |
| Ryobi ONE+ 18V battery, 4Ah or larger | 1+ | Genuine Ryobi only. Third-party "compatible" packs may lack protection circuitry. A 4Ah pack holds about 72Wh, which is close to a 5Ah 4S LiPo. |
| Ryobi ONE+ charger | 1 | Charge packs off the rover. |
| ONE+ battery adapter | 1 | Print one, e.g. the [Ryobi 18V Battery Receiver with Keystone 209 Clips](https://makerworld.com/models/2586809) (print in PETG or ABS rather than PLA), or buy a "Power Wheels" style adapter such as [this one](https://vxb.com/fr/products/power-wheels-adapter-for-ryobi-18v-one-plus-batter). Ignore the current ratings in seller listings; they are inconsistent. **We bought:** a "DIY battery dock / Power Wheels adapter for Ryobi 18V" on eBay (about $15–30). These often come with a 30A fuse: replace it with the 15A fuse below. Check polarity with a multimeter before connecting, and leave any built-in switch on (the board has its own main switch). |
| Keystone 209 spring clips | 2 | Only if you print the adapter. |
| 12–14AWG stranded wire, red and black | ~0.5m each | |
| Inline fuse holder + 15A fuse | 1 | Goes right at the adapter. The board's 10A fuse doesn't protect the wire between the battery and the board. |
| Low-voltage cutoff (LVC) module | 1 | See [Low-voltage cutoff](#low-voltage-cutoff). Needs at least 15A continuous, an adjustable or ~15V threshold, and low standby current. **We bought:** an XH-M609 on eBay; see the notes under [Low-voltage cutoff](#low-voltage-cutoff). |
| XT30 connector (MH-FC) | 1 | Same battery-side connector as in [extra parts](extra_parts.md). |
| Battery holder / mount | 1 | Custom or 3D printed. It must hold the pack and adapter firmly so the clips can't vibrate loose. |

**You no longer need:** the LiPo battery, the B6 balance charger, or the LiPo fire-safe bag from [extra parts](extra_parts.md).

### Wiring

```
Ryobi ONE+ pack
   │ (+)                                 (−)
   ▼                                      │
 adapter ── 15A fuse ── LVC module ── XT30 (MH-FC) ── board J1 (XT30-M)
                                          │
   (−) ───────────────────────────────────┘
```

Keep the fuse within a few centimeters of the adapter. Wire the LVC module according to its own instructions; most switch the negative or the positive lead with a MOSFET or relay.

### Build steps

1. **Build the adapter.** Solder the 12–14AWG leads to the clips *before* pressing them into a printed adapter, because soldering heat will warp the print. Slide a pack in and **check polarity with a multimeter**. Mark + and − on the adapter.
   > Note: a ONE+ pack may read only ~6V at its terminals with nothing connected. The pack switches its output on when it detects a load. This is normal. If the rover ever fails to power up from a pack, suspect this first.
2. **Add the fuse** on the positive lead, right at the adapter.
3. **Add the low-voltage cutoff** and set it to about **15V** (3.0V per cell). If you have a bench power supply, check that it disconnects when you lower the supply below the threshold and reconnects above it.
4. **Crimp or solder the XT30 (MH-FC)** at the end. Match the polarity to the silkscreen at J1 on the motor board.
5. **Mount the pack** on the rover so it can't move or pop off its clips while driving over rough terrain.

### Low-voltage cutoff

**This step is not optional.** Nothing on the rover protects the battery from over-discharge:
- The PCB has no undervoltage shutdown. The 5V regulator keeps running down to ~5.5V.
- The [rover software](https://github.com/nasa-jpl/osr-rover-code) only reports the battery voltage. It never stops anything because of it.
- The Roboclaw's Min Main Battery setting only stops the drive motors. The Raspberry Pi and servos keep draining the pack.

Ryobi packs have built-in protection, but its cutoff threshold isn't published. Some sellers say not to rely on it.

Options:
- **A ready-made LVC module** rated for at least 15A, set to ~15V. Pick one with low standby current, and remove the pack when the rover is stored.
- **XH-M609 (what we bought, from eBay).** A relay-based module with an LED voltage display. Listings give 12–36V input, a 20A relay and under 1.5W consumption; there's no manufacturer datasheet, so treat these as approximate.
  - Set the cutoff to **15.0V** and the reconnect voltage to about **16.5–17V**, so it doesn't flicker on and off when the pack voltage recovers after cutting off.
  - Before installing it, test it on the bench supply: lower the voltage and check it cuts off at 15.0V, then raise it and check it reconnects.
  - The relay coil stays powered while the rover runs. That's about 80mA at 18V, roughly 10% of runtime.
  - The display and control circuit stay on after a cutoff, so the pack keeps draining. Remove the pack after use.
  - Read the rating printed on the relay itself (the DC rating, not the AC one). Mount the module on foam tape or standoffs to limit vibration.
- **A DIY circuit**, e.g. this [battery low-voltage cutoff for Ryobi packs](https://hackaday.io/project/173963-battery-low-voltage-cutoff). It uses an op-amp comparator driving a MOSFET and cuts off at ~14.3V.

### Configure the Roboclaws

In Basicmicro Motion Studio, for **each** of the three Roboclaws (see [Testing the PCB and wiring for the drive motors](../electrical/pcb/README.md#6-testing-the-pcb-and-wiring-for-the-drive-motors)):

| Setting | Stock 4S value | ONE+ value | Why |
| --- | --- | --- | --- |
| Max Main Battery | 18.5V | **22V** | A full pack is ~21V. The stock value triggers "Main voltage High Error" and stops the motors. |
| Min Main Battery | 11.5V | **15V** | Stops the drive motors before the pack is over-discharged. |
| Max current (per channel) | default | **~5A** (≤7A) | Limits heating and stall current, which is higher at 21V. |

Click **Device > Write Settings** on each Roboclaw so the settings are saved.

### Configure the rover software

By default the rover code sends open-loop **duty** commands to the Roboclaws (`duty_mode: true` in `osr_bringup/config/roboclaw_params.yaml`). The average voltage a motor sees is therefore *duty × battery voltage*. The fastest command maps to roughly 68% duty with the default `velocity_qpps_to_duty_factor: 8`.

| Battery | Max duty | Max average motor voltage |
| --- | --- | --- |
| 4S LiPo, full (16.8V) | 68% | ~11.5V |
| ONE+, full (21V), factor 8 | 68% | **~14.3V** |
| ONE+, full (21V), factor **7** | 60% | ~12.6V |
| ONE+, nominal (18V), factor 7 | 60% | ~10.8V |

In `ROS/osr_bringup/config/roboclaw_params.yaml` of the [rover code](https://github.com/nasa-jpl/osr-rover-code), change:

```yaml
    velocity_qpps_to_duty_factor: 7  # was 8; lowered for a 5S (18V) battery
```

This keeps the motors near their 12V rating and keeps commanded speeds close to actual speeds. The factor must be an integer.

Alternatively, set `duty_mode: false` to use the Roboclaws' closed-loop velocity control. That only applies the duty each speed needs, but it depends on well-tuned velocity PID values on every Roboclaw.

### First power-up checklist

1. With the rover's main switch **off**, connect the battery lead to J1.
2. Switch on. Measure between test points T4 (`swt_out`) and T2 (`Batt-`). Expect ~18–21V depending on charge.
3. Check the 5V and 12V test points (see [Test all voltage lines](../electrical/pcb/README.md#310-test-all-voltage-lines)).
4. Run `scripts/roboclawtest.py` and `scripts/test_ina260.py` from the rover code. Confirm the battery voltage readings match your multimeter.
5. Confirm that Motion Studio shows no voltage errors on any Roboclaw.
6. Drive gently at first. Check motor temperature after a few minutes.

### Operating notes

- **Runtime:** expect roughly 2 hours of mixed driving and idling per 4Ah pack. This is scaled from the [3+ hours on a 7.2Ah 4S pack](../README.md) shown in the power usage chart, where the rover drew ~0.3A idle and ~1A while driving.
- **Use one pack at a time.** Never connect packs in parallel or series.
- **Remove the pack** when the rover is stored, so the LVC module and the electronics don't slowly drain it.
- **Charge off the rover** with the Ryobi charger.
- The main [README](../README.md) states that the rover "powers down below 13–14V". That doesn't apply here: nothing on the board enforces it, and a ONE+ pack hits its cutoff before reaching that range.

### References

- [Ryobi One+ battery teardown](https://www.sevarg.net/2017/05/21/ryobi-one-lithium-nicd-tool-battery/): protection IC and discharge FETs inside the pack, cells rated 15A continuous
- [Hackaday: Ryobi power packs as e-bike batteries](https://hackaday.com/2021/06/07/ryobi-power-packs-as-ebike-batteries/): community discussion of pack protection
- [Ryobi battery repair guide](https://badar.tech/2025/08/24/ryobi-battery-repair-guide/): load detection behavior
- [Battery low-voltage cutoff for Ryobi packs](https://hackaday.io/project/173963-battery-low-voltage-cutoff)
- [Pololu D24V150F5](https://www.pololu.com/product/2881), [Pololu D24V22F12](https://www.pololu.com/product/2855), [RoboClaw 2x7A](https://www.pololu.com/product/3682)

---

# Workspace

These aren't rover parts. They're what we chose to set up the build area. Prices are from the listings at the time of writing (October 2026) and may change.

## 2. Workbench mat: BSONEDO silicone mat, 24" x 80"

A full-length cover for a 24" x 79" desk. It protects the desktop from solder, flux, hot tools and scratches.

| Item | Size | Material | Link |
| --- | --- | --- | --- |
| BSONEDO Large Silicone Mat, black (also translucent or dark gray) | 24" x 80" | Silicone, cuttable | [Amazon](https://www.amazon.com/clp/B0FK4NLYH1) |

Notes:
- This is the only mat found in a 24" x 80" size. It overhangs a 79" desk by 1", or you can trim it with scissors or a utility knife.
- It's sold as a countertop/table mat. The listing doesn't give thickness or a maximum temperature, so check with the seller before relying on its heat resistance. Comparable silicone work mats are rated to about 445°F (230°C). Always put the soldering iron in its stand, never down on the mat.
- **It is not an ESD (anti-static) mat.** Handle the PCBs, Raspberry Pi and other boards on a grounded ESD mat or with a wrist strap. ESD rubber alternatives:
  - [RubberStat dual-layer ESD mat kit, 24" x 120"](https://www.amazon.com/clp/B0971BYTSJ), which can be cut down to 79"
  - [Rubber ESD soldering mat, 24" x 72", with ground cable](https://www.amazon.com/Rubber-Anti-Static-Temperature-Soldering-MAT-24/dp/B07DHWZNQQ)

## 3. Floor: MotoFloor Modular Garage Flooring Tiles

Interlocking 1' x 1' solid-top tiles from Costco. They give the work area a durable, easy-to-clean floor where dropped screws and nuts are easy to find.

| Item | Costco item # | Contents | Approx. price | Link |
| --- | --- | --- | --- | --- |
| MotoFloor Modular Garage Flooring Tiles (model MF48CHAR) | 2889845 | 48 click-lock tiles, 1' x 1' (48 sq ft). Colors: Charcoal, Black & White, Alloy, Black & Alloy. | $149.99 | [Costco](https://www.costco.com/MotoFloor-Modular-Garage-Flooring-Tiles-48-square-feet-per-Box-1%27-x-1%27-Tiles-.product.100009181.html) |
| MotoFloor Modular Garage Flooring Completer Kit | 617090 | 12 tiles + 20 beveled edges, 12.7mm (1/2") thick | $82.99 | [Costco](https://www.costco.com/motofloor-modular-garage-flooring-completer-kit.product.11038311.html) |

Notes:
- **Planning:** you need one tile box per 48 sq ft of floor, rounded up. Add completer kits for the beveled edge pieces along any side that isn't against a wall. Each kit has 20 edges, which presumably covers about 20 ft of exposed edge.
- **Don't mix with the self-draining tiles.** Costco also sells [MotoFloor Self-Draining Tiles](https://www.costco.com/motofloor-modular-self-draining-garage-floor-tiles.product.100566943.html), which are 3/4" thick and perforated. The completer kit's 1/2" edges are presumably meant for the solid modular tiles. Costco doesn't state compatibility, and it's unconfirmed whether the kit includes corner pieces.
- Online only, with a limit of 15 boxes per member. Delivery takes about 10–15 business days.
- The tiles snap together with no glue or fasteners. Lay them over a clean, dry floor.

## 4. Solder fume extractor: KOTTO Solder Smoke Absorber

A small benchtop fan with an activated-carbon filter. It pulls rosin flux smoke away from your face while you solder. This is enough for the roughly 20–30 soldering sessions the rover build takes.

| Item | Specs | Approx. price | Link |
| --- | --- | --- | --- |
| KOTTO Solder Smoke Absorber Remover Fume Extractor | 3000 RPM fan, 0.4" activated-carbon filter, ~5 x 3 x 5 in, ~2.2 lb, 120V / 30W | $30–40 | [Amazon search](https://www.amazon.com/s?k=KOTTO+solder+smoke+absorber+fume+extractor) |

Notes:
- **Placement matters most.** Keep the intake within about 5–6 in of the joint; smoke escapes upward beyond that.
- **Why not just an air purifier?** A room air purifier only cleans the air after the smoke has passed your face, so it doesn't replace an extractor at the joint. You can still run one, or open a window, as a supplement.
- Rosin flux fumes are a known cause of occupational asthma. The fumes come mostly from the flux, not the lead. Wash your hands after handling leaded solder.
- Replace the carbon filter regularly. Replacement filter packs are widely available.
- It's carbon only. If you have asthma, use leaded solder heavily, or plan to solder much more, consider an extractor with HEPA + carbon filters, such as the [FumeClear FC-2001S](https://www.fumeclear.com/blogs/buyer-s-guide-buying-advice/how-to-choose-soldering-fume-extractor) (~$90) or a Hakko FA-400 with its HEPA add-on.

## 5. Bench DC power supply: SPS-3010

An adjustable bench supply with current limiting. It powers the boards for testing without a battery, as the [PCB assembly guide](../electrical/pcb/README.md#34-test-the-power-connectors) does with "14.8V DC, current limit 1.0A". It also lets you simulate the ONE+ battery's 15–21V range and check that the low-voltage cutoff trips at about 15V (see [entry 1](#1-ryobi-one-18v-battery-power-supply)).

| Item | Output | Price | Link |
| --- | --- | --- | --- |
| DC Power Supply Variable, 0-30V 0-10A Adjustable Switching DC Regulated Bench Power Supply, model SPS-3010 (commonly sold under the Jesverty brand) | 0–30V, 0–10A, switching, with a 5V/2A USB port | about $50–90 | [Amazon search](https://www.amazon.com/s?k=SPS-3010+DC+power+supply+0-30V+0-10A), [Newegg](https://www.newegg.com/p/0FB-027H-00108) |

Notes:
- **Voltage range:** the 30V maximum covers everything the rover needs: 14.8V for the board tests, and up to ~21V for a full ONE+ pack. It doesn't reach 36V. For that you would need a 60V-class supply such as the OWON SPE6103.
- **Display and controls:** a 4-digit display showing voltage, current and power (0.01V / 0.001A / 0.1W), with coarse and fine adjustment knobs.
- **Constant voltage or constant current:** set the voltage and the current limit before connecting a load. The supply switches from constant-voltage to constant-current mode when the load reaches the limit. For the first power-up of an assembled board, start with a low limit such as 1.0A, as the PCB guide does.
- **Noise:** as a switching supply, it has more output noise than a linear supply. That's fine for powering the rover's boards, motors and regulators.
- **Check the model number:** similar-looking listings exist under several brands and model numbers. The SPS-3010M has a Type-C port, an encoder knob and an output on/off switch. The PS-3010DF is a different unit. Confirm it's the SPS-3010 before ordering.
- **One to avoid:** the FNIRSI IPS3608 (36V/8A). In [Hackaday's write-up of an independent review](https://hackaday.com/blog/page/18/?s=protocol), the unit overheated and locked up under load, and its measured ripple was ~200mV peak-to-peak against a claimed 10mV.
