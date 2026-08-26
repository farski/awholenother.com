---
layout: post
title: Electricity cost calculator
date: 2026-08-26 11:15 -0400
---

<input id=rate type=number step="0.1" min=0 max=1 value="0.3556" style="width: 200px; padding: 15px; font-size: var(--24px)"> $/kWh

<h2>Continuous</h2>

<input id=watt type=number step="0.1" min=0 max=2000 value=1000 style="width: 200px;"> W

<ul style="margin-block-end: 2em;">
	<li>$<span id=hour></span> per hour</li>
	<li>$<span id=day></span> per day</li>
	<li>$<span id=month></span> per 30d</li>
	<li>$<span id=year></span> per 365d</li>
	<li><span id=dollar></span> per dollar</li>
</ul>

<h2>Refill</h2>

<input id="kwh" type="number" step="0.1" min="0" max="500" value="18.8" style="width: 200px;"> kWh

<ul style="margin-block-end: 2em;">
	<li>$<span id=refill></span> per fill</li>
</ul>

<script style="display: none;" type="text/javascript">
	function money(amt) {
		if (amt.toString().includes(".")) {
			const d = amt.toString().split(".")[0];
			const c = amt.toString().split(".")[1];
			const cc = c.substring(0, 6).replace(/(0)+$/, "");
			return `${d}.${cc}`;
		} else {
			return amt;
		}
	}

	function calc(event) {
		const rate = document.getElementById("rate").value;

		// refill
		const kwh = document.getElementById("kwh").value;
		document.getElementById("refill").innerText = money(kwh * rate);

		// continuous
		const watt = document.getElementById("watt").value;

		const kw = watt / 1000;

		document.getElementById("hour").innerText = money(kw * rate);
		document.getElementById("day").innerText = money(kw * 24 * rate);
		document.getElementById("month").innerText = money(kw * rate * 24 * 30);
		document.getElementById("year").innerText = money(kw * rate * 24 * 365);

		const hours = 1 / (rate * kw);

		if (hours >= 1) {
			document.getElementById("dollar").innerText = `${
			Math.round(hours * 100) / 100
			} hours`;
		} else {
			const minutes = hours * 60;
			document.getElementById("dollar").innerText = `${
			Math.round(minutes * 100) / 100
			} minutes`;
		}
	}

	(function () {
		document.querySelectorAll("input").forEach((el) => {
			el.addEventListener("change", calc);
			el.addEventListener("keyup", calc);
		});

		calc();
	})();
</script>
