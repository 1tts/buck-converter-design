# Final Project Notes

## Design Spec

- 5V Input -> 1.8V @ 5A load buck converter PCB
	- $\Delta I_{L}<$ 30% of $I_{out}$
	- P2P ripple is less than 1% on $V_{out}$ at full load
	- P2P ripple is less than 50mV at full load
	- Greater than 50 degree of phase margin (?)
	- Less than 50mV transient droop when load steps from 2A -> 3A

Using the [[lm21215.pdf|TI LM21215]] Buck regulator

Inductor is to be chosen by us, based off of loss & various other calculations
All other components (input caps, output caps, compensation loop items) are to be selected by us as well. 0805 (0.08 in by 0.05in) surface mount components recommended.

## Design Flow

1. Hand Calculations - Calculate acceptable values for $L$, $C_{out}$, and $C_{in}$ based off of the 5V->1.8V & other conditions.

For example, we can get our value of L using the following; $$L = ((Vin-Vout)*Duty)/(\Delta I_{L}*fsw)$$
A crossover frequency of 30kHz to 50kHz and a phase margin > 50 degrees is necessary. 

**Crossover Frequency** refers to the frequency $\omega_{gc}$ where the loop gain is 0db. **Phase Margin** refers to the difference between actual phase and -180deg at that crossover frequency. Since the LM21215 is a 500kHz switching frequency, 10% of that would be 50kHz which makes sense. Having this phase margin ensures that the converters response to a transient load is fast enough so there isn't oscillation (i.e if there is significant phase lag, near -180deg, this means that by the time the circuit is able to react to the change in voltage the voltage may have already changed again, so instead of adjusting for that change you compound on top of it).

Equations for most components & losses are listed below:


> [!NOTE]+ Equations
> | Quantity                               | Equation                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Duty cycle                             | `D = Vout / Vin`                                                                                                   |
| Feedback resistor Rfb1                 | `Rfb1 = Rfb2 * (Vout / Vref - 1)`                                                                                  |
| Ripple current                         | `Delta_IL = ripple_pct * Iout_max`                                                                                 |
| Inductance                             | `L = ((Vin - Vout) * D) / (Delta_IL * fsw)`                                                                        |
| Peak current                           | `Ipeak = Iout_max + Delta_IL / 2`                                                                                  |
| Minimum output capacitance for ripple  | `Cout_min = Delta_IL / (8 * fsw * Vout_ripple_max)`                                                                |
| Minimum capacitance for droop          | `Cdroop_min = L * Istep^2 / (Vdroop_max * (Vin - Vout))`                                                           |
| DC conversion ratio                    | `dc = Vout / (Vin * efficiency)`                                                                                   |
| Minimum input capacitance current term | `Cin_min_current = (Iout_max * dc * (1 - dc)) / (fsw * Vin_ripple_max)`                                            |
| Input capacitor RMS current            | `Icin_rms = Iout_max * sqrt(D * (1 - D))`                                                                          |
| Output resistance                      | `Ro = Vout / Iout_max`                                                                                             |
| LC resonant frequency                  | `fLC = 1 / (2 * pi() * sqrt(L * Cout_eff))`                                                                        |
| Output capacitor ESR zero frequency    | `fESR = 1 / (2 * pi() * ESR * Cout_eff)`                                                                           |
| Rc1                                    | `Rc1 = (fc / fLC) * (Vramp / Vin) * Rfb1`                                                                          |
| Cc1                                    | `Cc1 = 1 / (pi() * fLC * Rc1)`                                                                                     |
| Cc2                                    | `Cc2 = Cc1 / (pi() * fsw * Rc1 * Cc1 - 1)`                                                                         |
| Rc2                                    | `Rc2 = Rfb1 * (fLC / (fESR - fLC))`                                                                                |
| Cc3                                    | `Cc3 = 1 / (2 * pi() * fESR * Rc2)`                                                                                |
| Inductor RMS current                   | `IL_rms = sqrt(Iout_max^2 + Delta_IL_target^2 / 12)`                                                               |
| Inductor conduction loss               | `P_inductor = IL_rms^2 * L_DCR`                                                                                    |
| High-side conduction loss              | `P_highside_cond = duty_cycle * (Iout_max^2 + Delta_IL_target^2 / 12) * Rds_highside`                              |
| Low-side conduction loss               | `P_lowside_cond = (1 - duty_cycle) * (Iout_max^2 + Delta_IL_target^2 / 12) * Rds_lowside`                          |
| Input capacitor RMS current (check)    | `Ic_in_rms_check = Iout_max * sqrt(duty_cycle * (1 - duty_cycle))`                                                 |
| Input capacitor ESR loss               | `P_cin_esr = Ic_in_rms_check^2 * Cin_ESR`                                                                          |
| Output capacitor RMS current           | `Icout_rms = Delta_IL_target / (2 * sqrt(3))`                                                                      |
| Output capacitor ESR loss              | `P_cout_esr = Icout_rms^2 * Cout_ESR_loss`                                                                         |
| Quiescent loss                         | `P_quiescent = Vin * I_quiescent`                                                                                  |
| Output power                           | `Pout = Vout * Iout_max`                                                                                           |
| Total power loss                       | `Ploss_total = P_inductor + P_highside_cond + P_lowside_cond + P_cin_esr + P_cout_esr + P_quiescent + Psw_assumed` |
| Experimental efficiency                | `Efficiency_exp = Pout / (Pout + Ploss_total)`                                                                     |
>
>

Once components are finalized based off calculations, recalculate to ensure that they still meet constraints (for example picked Inductor may not be exact calculated value, also need to ensure DCR still works w.r.t losses).

2. Verify AC & Transient behavior using SPICE
3. Finalize schematic, layout, and BOM
4. Fabricate PCB & order comps

## Layout Reqs.

- PCB: 3" x 3" max
- 0805 / 1210 Resistors/Inductors/Caps
- 2-layer only
- Inductor should have terminals on the side: ![[ENGR455 Lab Final Project Fall 26.pdf#page=5&rect=169,78,783,299|ENGR455 Lab Final Project Fall 26, p.5]]
## Rough Circuit Design

![[ENGR455 Lab Final Project Fall 26.pdf#page=8&rect=59,48,945,450|ENGR455 Lab Final Project Fall 26, p.8]]![[ENGR455 Lab Final Project Fall 26.pdf#page=9&rect=177,42,729,464|ENGR455 Lab Final Project Fall 26, p.9]]
### Feedback Resistors

Internal comparator compares input to output. Feedback resistor effectively is to divide ![[ENGR455 Lab Final Project Fall 26.pdf#page=11&rect=374,223,566,384|ENGR455 Lab Final Project Fall 26, p.11|300px|center]]
## Picking Capacitors

Use SimSurfing tool for Ceramic Capacitors

Say you want a 100uF capacitor, and say we want the cap rated at least twice our output voltage (3.6V). A lot of options - need to check the *DC Bias*, i.e when voltage changes on a side of a capacitor, the capacitance changes. When we do hand calculations, not going to be exactly 100uF, will likely be at some margin of error within 100uF. At higher voltage, bias may be much more significant, and will need to test different combinations of capacitors that are about ~100uF at 1.8V.

Generally speaking, at half the rated voltage, you'd get half the capacitance

## Picking Inductors

Pick shielded inductors