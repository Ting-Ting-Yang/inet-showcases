---
layout: page
title: Packet Loss vs. Distance Using Various WiFi Bitrates
---

## Goals

In this showcase we perform a parameter study examining how packet error rate
changes as a function of distance in a 802.11g wireless network.

Source files location: <a href="https://github.com/inet-framework/inet-showcases/tree/master/visualizer/earth" target="_blank"><var>inet/showcases/wireless/errorrate</var></a>
<br/>You can discuss this showcase <a href="https://github.com/inet-framework/inet-showcases/issues/7" target="_blank">on GitHub</a>.

<!--TODO:

    Hogy konfigurálod be az inetet, hogy ilyen parameter study-t tudj csinálni

    effective bitrate <- throughput DONE
    bele vannak számolva a bit hibák miatti veszteségek

    - hogy keletkezett ez a diagram...milyen statistisztikákat recordoltak, hogy kell beállítani a chart-ot
    - "a 10^-13-ont nullának tekintjük" DONE
    - #!!! ezeket el kell magyarázni mert essential
    - ahol lehet mást választani ott elmondani hogy mit lehet DONE
    - SNIRThreshold vs Sensitivity DONE
    - sensitivity: DONE
    - minden SNIR-rel kapcsolatos dolog szimulációs optimalizáció DONE
    - Packet loss vs. distance using various wifi bitrates DONE
    - legyen g(erp)-vel DONE
    - freespacepathloss legyen? -> inkább tworay DONE
    - kell a táblázat / összehasonlítás ? -> referenciaként be lehet linkelni...mérnöki hibahatáron belül van DONE
    - NIST error model ne a goals-ba DONE
-->
<!--
    TODO

    The Ieee80211NistErrorModel is based on the NIST error rate model, which is the default 802.11 error rate model in
    the ns-3 network simulator. URL HERE
    Which is based on [miller2003], done at NIST (Nation Institude of Standards and Technology)
    -->

## The Model

The network contains two hosts operating in 802.11g ad-hoc mode at 10 mW
transmission power. One of the hosts acts as traffic source, the other as traffic
sink. We will perform a parameter study with the distance and the bitrate as
parameters. The distance will run between 10 and 550 meters, in 2 meter steps.
The bitrate will take the ERP modes in 802.11g: 6, 9, 12, 18, 24, 36, 48 and 54
Mbps. This results in about 2100 simulation runs.

To make the model more realistic, we will simulate multipath propagation using
the <var>TwoRayGroundReflection</var> path loss model. (There are various
path loss models in INET, including <var>FreeSpacePathLoss</var>,
<var>RayleighFading</var>, <var>RicianFading</var>,
<var>LogNormalShadowing</var> and <var>NakagamiFading</var>). The two ray
ground reflection path loss model requires a ground model, which is configured in
the <var>physicalEnvironment</var> module to be <var>FlatGround</var>.
The heights of the hosts above the ground is set to 1.5 meters. We assume
isotropic background noise of -86 dBm (<var>IsotropicScalarBackgroundNoise</var>.)

We will use <var>Ieee80211NistErrorModel</var> to compute bit errors. The
<var>Ieee80211NistErrorModel</var> is based on the Nist error rate model.

In each simulation run, the source host will send a single UDP packet (56 bytes of
UDP data, resulting in a 120-byte frame) to the destination host as a probe. At
packet reception, the error model will compute bit error rate (BER) from the
signal-to-noise-plus-interference-ratio (SNIR). Packet error rate (PER) will be
computed from BER. The simulation records SNIR, BER, and PER. Note that in this
simulation model, the physical layer simulation is entirely deterministic, hence
there is no need for Monte Carlo.

<!--
- the snir threshold is important because when the transmission's snir is below this value, the reception is not simulated because it is assumed to be incorrect.
- thus we have to set the snir threshold low enough to get more fine-grained data at low snir levels
-->

We lower the <var>snirThreshold</var> parameter of the hosts' radios from 4 to
0 dB. When the transmission's SNIR is below this value, the reception is assumed
to be incorrect, and the reception is not simulated. Lowering the level results in
more fine-grained data at low SNIR levels.

## Results

SNIR, BER and PER are recorded from the simulation runs. SNIR is measured at the
receiver, and depends on the power of the noise and the power of the signal. The
signal power decreases with distance, so SNIR does as well. SNIR is independent
of modulations and coding schemes, so it is the same for all bitrates. This can be
seen on the following plot, which displays SNIR against distance.

<img src="SNIR_distance_v2.png" class="screen" />

The next plot shows how packet error rate decreases with SNIR. Slower bitrates
use simpler modulation like binary phase shift keying, which is more tolerant to
noise than more complex modulations used by faster bitrates, hence the
difference on the graph between the different bitrates.

The various modulations and coding rates of 802.11g ERP modes are the
following:

- 6 and 9 Mbit/s modes use BPSK, coding rates 1/2 and 3/4
- 12 and 18 Mbit/s modes use QPSK, coding rates 1/2 and 3/4
- 24 and 36 Mbit/s modes use 16-QAM, coding rates 1/2 and 3/4
- 48 and 54 Mbit/s modes use 64-QAM, coding rates 1/2 and 3/4

Note that the completely flat and completely vertical lines on the plot are due to
the signal power decreasing below the sensitivity of the receiver.

<img src="PER_SNIR_v3.png" class="screen" />

The following plot shows the packet error rate vs distance. Again, slower bitrates
show less packet errors as the distance increases because of the simpler
modulation.

<img src="PER_distance_v3.png" class="screen" />

We also compute the effective bitrate, which is the gross bitrate decreased by
packet errors. It is computed with the following formula:

`effective bitrate = (1-PER) * nominal bitrate`

It is equal to the nominal data bitrate unless it is decreased because of bit errors
as the distance increases.

Effective bitrate vs distance is shown on the next plot. Higher bitrates are more
sensitive to increases in distance, as the effective bitrate drops rapidly after a
critical distance. This critical distance is farther for slower bitrates, and the
decrease is not as rapid.

<img src="throughput_distance3.png" class="screen" />

802.11 ranges depend on many variables, e.g. transmission power, receiver
sensitivity, antenna gains and directionality, and background noise levels. The
above ranges correspond to arbitrary values for the variables. In reality ranges
can vary significantly.

## Conclusion

Packet error rate increases quickly as the distance approaches the critical point.
Slower bitrates are less sensitive to increasing distance, because they use simpler
modulation. Faster bitrate modes are advantageous in short distances because of
the increased throughput, but slower modes work better at longer distances.
Furthermore, using rate adaptation, a host can use fast modes for short distances
and slower modes for larger ones. When the number of lost packets increases and
throughput drops, it becomes more viable to change to a slower bitrate mode. For
example, the rate control algorithm could change to the slower bitrate at around
the critical point, about where the curves for two adjacent bitrate modes intersect.

## Further Information

More information can be found in the <a href="https://omnetpp.org/doc/inet/api-current/neddoc/index.html" target="_blank">INET Reference</a>.

## Discussion

Use <a href="https://github.com/inet-framework/inet-showcases/issues/7" target="_blank">this page</a>
in the GitHub issue tracker for commenting on this showcase.
