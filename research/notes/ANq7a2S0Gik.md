# Chris Rosser — "Stop Killing ESCs: The Ultimate Capacitor Buying Guide"
https://youtu.be/ANq7a2S0Gik

No bench-test rig for this video — a technical/buying-guide explainer, illustrated with thermal-camera footage from the creator's other ESC testing.

## Capacitor lead material (new finding, not previously in this KB)
- Capacitor leads are usually copper-clad steel, not solid copper — steel core plated in copper, then nickel.
- Graded by IACS (International Annealed Copper Standard): resistance relative to pure copper of the same length/diameter. Cheap/unbranded caps ≈ 20% IACS (5x the resistance of copper). Higher-grade branded caps: 30-40% IACS.
- At ESC switching currents, I²R heating in a low-IACS lead raises its own resistance further → thermal runaway → lead can melt and separate from the ESC, killing filtering entirely at the worst possible moment.
- IACS grade can't be judged by look/feel/magnetic response — only confirmable via thermal-camera testing under load or a manufacturer datasheet.

## Capacitor construction types
- **Wet electrolytic:** aluminum foil anode + Al-oxide dielectric + liquid-electrolyte-soaked paper cathode. Ions bump through liquid → higher ESR. Electrolyte can regrow the oxide layer after a moderate overvoltage spike (self-healing).
- **Solid polymer:** same anode/dielectric, but solid conductive polymer cathode (electrons, not ions) → ESR 1,000-10,000x lower than wet electrolytic. Cannot self-heal — an overvoltage breakdown melts the polymer and the cap fails short/explosively. More fragile against spikes at the same voltage rating.
- **Hybrid polymer:** solid polymer cathode + a small amount of liquid electrolyte for partial self-healing. Best of both, higher cost.
- Wet electrolytic loses 20-50% of rated capacitance at 24-48kHz (rated at 120Hz) — ions can't fully penetrate the foil's etched surface area at high frequency. Solid/hybrid polymer retains capacitance much better at switching frequency (electrons move faster than ions).

## Sizing
- Voltage rating: 1.5x-2x max pack voltage (1.5x smaller/cruising builds, 2x heavier/aggressive builds or extra crash margin).
- Capacitance by prop size: ≤2.5" → 220µF; ≤4" → 330-470µF (hybrid polymer preferred on 4S/6S); 5" → 680-1000µF wet electrolytic or 470-680µF hybrid polymer (explicitly recommends against solid polymer at 5"+ — seen failing in testing); 7" → 1000-2000µF wet electrolytic; 8"-10" → 2000µF+; 10"+ → 3000µF+.
- For large builds needing 2000-4000µF: prefer several smaller caps in parallel over one large cap — easier to fit, lower net ESR (four 1000µF in parallel ≈ 1/4 the ESR of one 1000µF).

## Brand/series recommendations (direct quotes, auto-caption transcript)
- Wet electrolytic: **Panasonic FR** (recommended default, modern/small/durable) or **FM** (max performance, physically larger, shorter service life). **Rubycon ZLJ/ZLH** (recommended); **FS/ZLQ** more compact but lower ripple-current capacity.
- Solid polymer: transcript renders the brand as "Unicorn"/"Unicon" — this is almost certainly **Nichicon** (auto-caption mishearing of a real, verifiable brand); their **UPL** series is cited as increasingly common because iFlight ships it stock on their ESCs.
- Hybrid polymer: Panasonic, Rubycon, and (the same "Unicorn"/Nichicon) all make hybrid-polymer ranges — recommended for max performance/durability, at a price premium.
- Explicitly named to avoid: **JWCO, Chong X/Cheng X, CapXon** — cheap brands using low-grade copper-clad-steel leads; observed melting at mid-throttle and capacitor bodies exceeding 120°C. Any unbranded "low ESR"-marked capacitor: avoid.

## Not covered
- No TVS diode testing in this video (see VgHOHcWu7-U for that).
