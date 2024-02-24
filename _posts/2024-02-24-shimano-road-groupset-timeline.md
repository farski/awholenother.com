---
layout: post
title: Shimano road groupset timeline
date: 2024-02-24 01:11 -0500
tags:
  - Bicycles
  - Cycling
  - Shimano
---

<style>
	table {
		border-collapse: separate;
		border-spacing: 0;
		margin-left: 50%;
		table-layout: fixed;
		transform: translateX(-50%);
		width: 100vw;
		margin-top: 20px;

		--sp6: firebrick;
		--sp7: navy;
		--sp8: slategray;
		--sp9: aquamarine;
		--sp10: tan;
		--sp11: #C7B6DC;
		--sp12: silver;
	}

	table tbody tr {
		margin: 10px;
	}

	table th:first-child { width: 100px; }
	table td {
		--mech-c1: rgba(0, 0, 0, 0);
		--mech-c2: rgba(255, 255, 255, 0.66);
		--mech-combo: 100%;

		--brake-c1: rgba(0, 0, 0, 0);
		--brake-c2: rgba(0, 0, 0, 0);

		font-size: 0.8em;
		position: relative;
		border-image: repeating-linear-gradient(
			90deg,
			var(--brake-c1),
			var(--brake-c1) 6px,
			var(--brake-c2) 6px,
			var(--brake-c2) 12px
		) 6;
		border-width: 7px;
		border-top-width: 0;
		border-left-width: 0;
		border-right-width: 0;
	}

	table td::before {
		content: '';
		top: var(--mech-combo);
		right: 0;
		bottom: 0;
		left: 0;
		position: absolute;
		background-image: linear-gradient(45deg, var(--mech-c1) 25%, var(--mech-c2) 25%, var(--mech-c2) 50%, var(--mech-c1) 50%, var(--mech-c1) 75%, var(--mech-c2) 75%, var(--mech-c2) 100%);
background-size: 30px 30px;
		z-index: 1;
	}

	table td span {
		font-weight: bold;
		position: relative;
		z-index: 2;
	}

	[mechanical] {}
	[di2][mechanical] { --mech-combo: 50%; }
	[di2] { --mech-combo: 0; }

	[rim] {
		--brake-c1: black;
		--brake-c2: black;
	}
	[disc] {
		--brake-c1: OrangeRed;
		--brake-c2: OrangeRed;
	}
	[rim][disc] {
		--brake-c1: black;
		--brake-c2: OrangeRed;
	}

	[sp6] {
		background: var(--sp6);
		color: white;
	}

	[sp7] {
		background: var(--sp7);
		color: white;
	}

	[sp8] {
		background: var(--sp8);
		color: white;
	}

	[sp9] {
		background: var(--sp9);
	}

	[sp10] {
		background: var(--sp10);
	}

	[sp11] {
		background: var(--sp11);
	}

	[sp12] {
		background: var(--sp12);
	}
</style>

**Last updated February 2024.**

Numbering inconsitencies:

- Dura-Ace begain with `7xxx` series numbers, but switched to `9xxx` after 7900, and maintained the sequence even once including the "R" prefix.
- Ultegra began with `6xxx` series numbers, but switched to `8xxx` at the same time it adopted an "R" prefix. (Ignoring the original 600 series naming.)
- 105 began with `5xxx` series numbers, but switched to `7xxx` at the same time it adopted an "R" prefix.
- Sora and Claris have always used `3xxx`/`2xxx` series numbers, but reset from 3500/2400 down to R3000/R2000 when they adopted the "R" prefix.
- For the latest generations, `xx00` indicates mechanical shifting, and `xx50` indicates Di2. Numbers are increased by 20 to indicate disc brake components (e.g., `xx20` for mechanical shifting and disc brakes, and `xx70` for Di2 and disc brakes). But some early Di2 components used `xx70` to distinguish between mechanical and electronic shifting, even when only compatible with rim brakes.

<hr>

Solid background = mechanical shifting, striped = Di2.
<br><span style="border-bottom: 3px solid black;">Black border</span> = rim brakes, <span style="border-bottom: 3px solid OrangeRed">orange</span> = disc brakes.

<div style="display: inline-block; padding: 6px 10px; background: firebrick; border-radius: 4px; color: white;">6</div>
<div style="display: inline-block; padding: 6px 10px; background: navy; border-radius: 4px; color: white;">7</div>
<div style="display: inline-block; padding: 6px 10px; background: slategray; border-radius: 4px; color: white;">8</div>
<div style="display: inline-block; padding: 6px 10px; background: aquamarine; border-radius: 4px;">9</div>
<div style="display: inline-block; padding: 6px 10px; background: tan; border-radius: 4px;">10</div>
<div style="display: inline-block; padding: 6px 10px; background: #C7B6DC; border-radius: 4px;">11</div>
<div style="display: inline-block; padding: 6px 10px; background: silver; border-radius: 4px;">12 speed</div>

<table>
	<thead>
		<tr>
			<th></th>
			<th colspan=6>1974</th>
			<th colspan=5>‘80</th>
			<th colspan=5>‘85</th>
			<th colspan=5>‘90</th>
			<th colspan=5>‘95</th>
			<th colspan=5>2000</th>
			<th colspan=5>‘05</th>
			<th colspan=5>‘10</th>
			<th colspan=5>‘15</th>
			<th colspan=5>‘20</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Dura-Ace</td>
			<td mechanical rim colspan=4></td>
			<td mechanical rim colspan=2><span>7200</span></td>
			<td mechanical rim colspan=4><span>7300</span></td>
			<td sp6 mechanical rim colspan=3><span>7400</span></td>
			<td sp7 mechanical rim><span>7400</span></td>
			<td sp8 mechanical rim colspan=8><span>7400</span></td>
			<td sp9 mechanical rim colspan=8><span>7700</span></td>
			<td sp10 mechanical rim colspan=4><span>7800</span></td>
			<td sp10 mechanical rim colspan=1><span>7900</span></td>
			<td sp10 mechanical di2 rim colspan=3></td>
			<td sp11 mechanical di2 rim colspan=4><span>9000</span></td>
			<td sp11 mechanical di2 rim disc colspan=6><span>9100</span></td>
			<td sp12 di2 rim disc colspan=3><span>R9200</span></td>
		</tr>
		<tr style="height: 10px;"></tr><tr style="height: 10px;"></tr>
		<tr>
			<td>Ultegra</td>
			<td mechanical rim colspan=4><span>600</span></td>
			<td mechanical rim colspan=3><span>6200</span></td>
			<td mechanical rim colspan=6><span>6300</span></td>
			<td sp7 mechanical rim colspan=10><span>6400</span></td>
			<td sp9 mechanical rim colspan=8><span>6500</span></td>
			<td sp10 mechanical rim colspan=4><span>6600</span></td>
			<td sp10 mechanical rim colspan=2><span>6700</span></td>
			<td sp10 mechanical di2 rim colspan=2></td>
			<td sp11 mechanical di2 rim colspan=4><span>6800</span></td>
			<td sp11 mechanical di2 rim disc colspan=5><span>R8000</span></td>
			<td sp12 di2 rim disc colspan=3><span>R8100</span></td>
		</tr>
		<tr style="height: 10px;"></tr><tr style="height: 10px;"></tr>
		<tr>
			<td>105</td>
			<td colspan=24></td>
			<td sp9 mechanical rim colspan=7><span>5500</span></td>
			<td sp10 mechanical rim colspan=5><span>5600</span></td>
			<td sp10 mechanical rim colspan=4><span>5700</span></td>
			<td sp11 mechanical rim colspan=4><span>5800</span></td>
			<td sp11 mechanical rim disc colspan=5><span>R7000</span></td>
			<td sp12 di2 disc colspan=1><span>R7100</span></td>
			<td sp12 mechanical di2 disc colspan=1></td>
		</tr>
		<tr style="height: 10px;"></tr><tr style="height: 10px;"></tr>
		<tr>
			<td>Tiagra</td>
			<td colspan=26></td>
			<td sp9 mechanical rim colspan=6><span>4400</span></td>
			<td sp9 mechanical rim colspan=5><span>4500</span></td>
			<td sp10 mechanical rim colspan=4><span>4600</span></td>
			<td sp10 mechanical rim colspan=4><span>4700</span></td>
			<td sp10 mechanical rim disc colspan=6></td>
		</tr>
		<tr style="height: 10px;"></tr><tr style="height: 10px;"></tr>
		<tr>
			<td>Sora</td>
			<td colspan=26></td>
			<td sp8 mechanical rim colspan=7><span>3300</span></td>
			<td sp9 mechanical rim colspan=5><span>3400</span></td>
			<td sp9 mechanical rim colspan=4><span>3500</span></td>
			<td sp9 mechanical rim colspan=9><span>R3000</span></td>
		</tr>
		<tr style="height: 10px;"></tr><tr style="height: 10px;"></tr>
		<tr>
			<td>Claris</td>
			<td colspan=29></td>
			<td mechanical rim colspan=6><span>2200</span></td>
			<td sp8 mechanical rim colspan=4><span>2300</span></td>
			<td sp8 mechanical rim colspan=4><span>2400</span></td>
			<td sp8 mechanical rim colspan=8><span>R2000</span></td>
		</tr>
	</tbody>
</table>
