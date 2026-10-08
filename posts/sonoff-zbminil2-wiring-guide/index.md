# Sonoff ZBMINIL2 Wiring Guide: No Neutral, Keep the Existing Wall Switch

> Wire a Sonoff ZBMINIL2 with no neutral and keep the existing wall switch: terminals, the sleeved blue wire, pairing with Home Assistant and what to check first.

Source: https://exitcode0.net/posts/sonoff-zbminil2-wiring-guide/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-09-26
Updated: 2026-10-08
Tags: sonoff, zigbee, home-automation, home-assistant

> **Updated October 2026:** added the direct answer, a terminal table, the neutral question, switch types and pairing, a troubleshooting section and the agent handoff block. The video and the original diagram are unchanged.

The [Sonoff ZBMINIL2](https://amzn.to/3N4tXwE) (Amazon) is a Zigbee relay that fits behind a UK light switch and needs no neutral at the switch. Wire it in series with the switched live and connect the existing rocker switch to S1 and S2, and the light works from the wall switch, from Home Assistant, or both. The whole job is four terminals, and the only trap is that the blue wire in the switch drop is a live, not a neutral.

**If you are unsure of what you are doing at any point, please contact a qualified electrician and do not attempt any electrical circuit modifications without properly isolating and de-energizing the circuit**

## Video Guide

Video: https://www.youtube.com/watch?v=Jgp4Ue_Gb-E

## Wiring Diagram

![Wiring diagram for the Sonoff ZBMINIL2, retaining the dumb switch](https://exitcode0.net/images/sonoff-zbminil2-wiring-diagram.jpg)

## Terminals

| Terminal | Connects to | Wire in a typical UK switch drop |
|---|---|---|
| L in | Permanent live from the switch feed | Brown |
| L out | Switched live to the light | Blue, over-sleeved brown |
| S1 | Common terminal of the existing switch | Short link wire |
| S2 | L1 terminal of the existing switch | Short link wire |

The ZBMINIL2 sits where the switch used to break the circuit. The existing switch no longer carries the lighting load at all. It only tells the relay to toggle through S1 and S2.

### Wiring Breakdown

**S1 and S2 terminals**

- The S1 terminal of the Sonoff ZBMINIL2 is connected to the common terminal of the dumb switch.
- The S2 terminal is connected to the L1 terminal (typically the live or load side) of the dumb switch.

**Switch feed**

- A cable from the switch feed supplies power to the Sonoff ZBMINIL2 and is linked to the dumb switch.

## Does the ZBMINIL2 need a neutral?

No. That is the whole point of the L2 over the original ZBMINI. It powers itself through the lighting load, so it works in a UK switch back box that only has a live loop in and a switched live out. The trade-off is the load. Very low-wattage LED bulbs can glow faintly or flicker when the relay is off, because a small current is still flowing to keep the relay powered. Sonoff sells an anti-flicker module for that case, or you can fit a bulb with a slightly higher wattage. My bathroom LED fitting has been fine without one.

## Explanation

This configuration lets the dumb switch operate the light manually while the Sonoff ZBMINIL2 handles the smart control.

In a "no neutral" smart switch setup, the blue wire is typically the neutral wire in electrical systems. However, in a no-neutral setup, the blue wire is re-purposed as a live conductor, which can be potentially confusing and dangerous if someone assumes it's still a neutral wire.

That is why the blue wire gets sleeved. In the UK, BS 7671 (the wiring regulations) requires a conductor used as a line to be identified as such, so a blue core doing the job of a switched live gets brown sleeving or brown tape at both ends. It tells the next person into that back box that the blue is live, it is what any electrician will expect to find, and it is the difference between a tidy job and a hazard waiting for someone else.

## Rocker switch or push button

Out of the box the ZBMINIL2 expects a rocker (toggle) switch on S1 and S2, so any change of state toggles the relay. That is what makes the existing wall switch keep working. If you'd rather use a retractive push button, change the switch type in Zigbee2MQTT or ZHA after pairing. In Zigbee2MQTT it is the `switch_type` setting on the device page. In ZHA it is under the device's configuration tab.

## Two-way (two switch) circuits

I've only wired the single-switch setup shown in the video. Two-way circuits, where the same light is controlled from two switches with strappers between them, come up constantly in the forum threads about this relay and I'm not going to guess at a diagram for a job I haven't done. The thing to understand is that the ZBMINIL2 toggles on any change at S1 and S2, so the existing two-way switches can stay in place as long as the relay sees a change when either one is flipped. Sonoff's own ZBMINIL2 documentation has the two-way drawing. When I've done one myself, it'll get its own section here.

## Pairing with Home Assistant

1. Put your coordinator into pairing mode (Zigbee2MQTT: Permit join. ZHA: Add device).
2. Power the circuit on. A brand new ZBMINIL2 goes straight into pairing mode. An already-paired one needs the button on the relay held for about five seconds until the LED blinks.
3. Give it a name that says where it is, not what it is. "Bathroom light" beats "ZBMINIL2 3".
4. Toggle it once from Home Assistant and once from the wall switch to confirm both paths work.

Because it is a mains-powered device it also acts as a Zigbee router, which helps the mesh reach battery sensors nearby. If you're short on routers elsewhere, [ESPHome Bluetooth proxies](https://exitcode0.net/posts/home-assistant-with-esphome-bluetooth-proxies/) solve the same reach problem for Bluetooth devices.

## Troubleshooting

**The light glows or flickers when switched off**
The LED load is below what the relay needs to power itself. Fit Sonoff's anti-flicker module across the fitting or use a bulb with a higher wattage.

**The wall switch toggles the light twice, or does nothing**
The switch type doesn't match the switch. Set `switch_type` to toggle for a rocker or momentary for a push button, and check S1 and S2 are on the common and L1 terminals rather than L1 and L2.

**It paired but keeps dropping off the mesh**
Metal back boxes and a long way to the coordinator do that. Add a router somewhere between, which is another mains Zigbee device, and re-pair.

**Home Assistant sees it but the light doesn't respond**
Check L in and L out aren't swapped. The relay will usually still power up, but the light won't switch.

---

I published the video accompanying this post over on the [DataSolace](https://datasolace.com/) YouTube channel - https://www.youtube.com/@DataSolace. We are producing all kinds of SmartHome and Network related content over there, so if this piqued your interest, consider heading over there to see more.

## Or just hand this page to your agent

The wiring is yours to do, but the Home Assistant side can be delegated:

```text
Read https://exitcode0.net/posts/sonoff-zbminil2-wiring-guide/ and help me pair a Sonoff ZBMINIL2 with Home Assistant and set the switch type to match my wall switch. Before changing anything, check whether I'm running Zigbee2MQTT or ZHA, show me the device list, and ask me which switch type I have. Do not give electrical wiring instructions beyond what the page says; that part is done by a person.
```

The page gives the agent the pairing steps and the setting names. Only you know which coordinator you run, what the switch is called and whether the wiring is finished.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/sonoff-zbminil2-wiring-guide/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
