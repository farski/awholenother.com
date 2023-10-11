---
layout: post
title: iOS support timeline
date: 2023-01-15 07:21 -0400
tags:
  - Apple
  - iOS
  - iPhone
  - iPad
---

<style>
	table {
		border-collapse: collapse;
		border-spacing: 0;
		margin-left: 50%;
		table-layout: fixed;
		transform: translateX(-50%);
		width: 100vw;

		--ios-1: #62d446;
		--ios-2: #d13521;
		--ios-3: #06b6ff;
		--ios-4: #737373;
		--ios-5: #6acad5;
		--ios-6: #203645;
		--ios-7: #f5545f;
		--ios-8: #ffc835;
		--ios-9: #58ddb1;
		--ios-10: #038df7;
		--ios-11: #e3bea8;
		--ios-12: #ff1db8;
		--ios-13: #fda813;
		--ios-14: #3e5997;
		--ios-15: #fb6501;
		--ios-16: #006ed9;
		--ios-17: #df3021;
	}

	table tr:hover { background: #eca99a; }

	table th:first-child { width: 145px; }
	table th:nth-child(2) { width: 15px; }
	table td { padding: 0; border: 0; }
	table td:first-child { padding: 3px 8px; }

	table td[data-version]::after {
		content: attr(data-version);
		font-weight: bold;
		padding: 5px;
	}
	table td[data-version][v6]::after, table td[data-version][v14]::after { color: white; }

	[v1] { background: var(--ios-1); }
	[v2] { background: var(--ios-2); }
	[v3] { background: var(--ios-3); }
	[v4] { background: var(--ios-4); }
	[v5] { background: var(--ios-5); }
	[v6] { background: var(--ios-6); }
	[v7] { background: var(--ios-7); }
	[v8] { background: var(--ios-8); }
	[v9] { background: var(--ios-9); }
	[v10] { background: var(--ios-10); }
	[v11] { background: var(--ios-11); }
	[v12] { background: var(--ios-12); }
	[v13] { background: var(--ios-13); }
	[v13][v14] { background: linear-gradient(180deg, var(--ios-13) 0%, var(--ios-13) 50%, var(--ios-14) 50%, var(--ios-14) 100%); }
	[v14] { background: var(--ios-14); }
	[v14][v15] { background: linear-gradient(180deg, var(--ios-14) 0%, var(--ios-14) 50%, var(--ios-15) 50%, var(--ios-15) 100%); }
	[v15] { background: var(--ios-15); }
	[v15][v16] { background: linear-gradient(180deg, var(--ios-15) 0%, var(--ios-15) 50%, var(--ios-16) 50%, var(--ios-16) 100%); }
	[v16] { background: var(--ios-16); }
	[v16][v17] { background: linear-gradient(180deg, var(--ios-16) 0%, var(--ios-16) 50%, var(--ios-17) 50%, var(--ios-17) 100%); }
	[v15][v16][v17] { background: linear-gradient(180deg, var(--ios-15) 0%, var(--ios-15) 25%, var(--ios-16) 25%, var(--ios-16) 62%, var(--ios-17) 62%, var(--ios-17) 100%); }
	[v17] { background: var(--ios-17); }

	[unsupported] td { border-top: 4px solid red; }
	[discontinued] td { border-top: 4px solid green; }

	[prod] { border-bottom: 2px solid black; }
	[prod] + td:not([prod]) { border-left: 4px solid black; }

	[nosdk] { background: repeating-linear-gradient(
    45deg,
    #606dbc,
    #606dbc 10px,
    #465298 10px,
    #465298 20px
  ); }
</style>

**Last updated September 2023.**

Black lines indicate the production period. Colored areas indicate the support period for a major iOS version on a given device. Striped bands show when a device's newest OS was no longer compatible with apps submitted to the App Store (WIP).

Models above the red line run only unsupported versions of iOS. Models below the green line run the latest major iOS version. Those between the lines run a supported version, but not the latest version.

<table>
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th colspan=12>2007</th>
			<th colspan=12>2008</th>
			<th colspan=12>2009</th>
			<th colspan=12>2010</th>
			<th colspan=12>2011</th>
			<th colspan=12>2012</th>
			<th colspan=12>2013</th>
			<th colspan=12>2014</th>
			<th colspan=12>2015</th>
			<th colspan=12>2016</th>
			<th colspan=12>2017</th>
			<th colspan=12>2018</th>
			<th colspan=12>2019</th>
			<th colspan=12>2020</th>
			<th colspan=12>2021</th>
			<th colspan=12>2022</th>
			<th colspan=10>‘23</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>iPhone</td>
			<td title=ARMv6>"A1"</td>
			<!-- 2007 -->
			<td></td><td></td><td></td><td></td><td></td><td v1 prod colspan=7 data-version=1></td>
			<!-- 2008 -->
			<td v1 prod></td><td v1 prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod v2></td><td v2 colspan=5 data-version=2></td>
			<!-- 2009 -->
			<td v2></td><td></td><td></td><td></td><td></td><td v3 colspan=7 data-version=3></td>
			<!-- 2010 -->
			<td v3></td><td v3></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<!-- 2011 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<!-- 2012 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<td nosdk colspan=134></td>
		</tr>
		<tr>
			<td>iPhone 3G</td>
			<td title=ARMv6>"A2"</td>
			<!-- 2007 -->
			<td colspan=12></td>
			<!-- 2008 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td v2 prod></td><td v2 prod></td><td v2 prod></td><td v2 prod></td><td v2 prod></td><td v2 prod></td>
			<!-- 2009 -->
			<td v2 prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td>
			<!-- 2010 -->
			<td prod v3></td><td prod v3></td><td prod></td><td prod></td><td prod></td><td prod v4></td><td v4 colspan=5 data-version=4></td><td></td>
			<!-- 2011 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<!-- 2012 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<td nosdk colspan=134></td>
		</tr>
		<tr>
			<td>iPhone 3GS</td>
			<td title=ARMv7>"A3"</td>
			<td colspan=24></td>
			<!-- 2009 -->
			<td></td><td></td><td></td><td></td><td></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td><td prod v3></td>
			<!-- 2010 -->
			<td prod v3></td><td prod v3></td><td prod></td><td prod></td><td prod></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod></td>
			<!-- 2011 -->
			<td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod v5 colspan=3 data-version=5></td>
			<!-- 2012 -->
			<td prod v5></td><td prod v5></td><td prod v5></td><td prod v5></td><td prod v5></td><td prod></td><td prod></td><td prod></td><td prod v6></td><td v6 colspan=3 data-version=6></td>
			<!-- 2013 -->
			<td v6 colspan=12></td>
			<!-- 2014 -->
			<td v6></td><td v6></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<td colspan=64></td>
			<td nosdk colspan=42></td>
		</tr>
		<tr>
			<td>iPhone 4</td>
			<td title=ARMv7>A4</td>
			<td colspan=36></td>
			<!-- 2010 -->
			<td></td><td></td><td></td><td></td><td></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td>
			<!-- 2011 -->
			<td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod v4></td><td prod></td><td prod></td><td prod v5></td><td prod v5></td><td prod v5></td>
			<!-- 2012 -->
			<td v5></td><td v5></td><td v5></td><td v5></td><td v5></td><td></td><td></td><td></td><td v6></td><td v6></td><td v6></td><td v6></td>
			<!-- 2013 -->
			<td v6></td><td v6></td><td v6></td><td></td><td></td><td></td><td></td><td></td><td v7 colspan=4 data-version=7></td>
			<!-- 2014 -->
			<td v7></td><td v7></td><td v7></td><td v7></td><td v7></td><td v7></td><td></td><td></td><td></td><td></td><td></td><td></td>
			<!-- 2015 -->
			<td colspan=64></td>
			<td nosdk colspan=42></td>
		</tr>
		<tr>
			<td>iPhone 4S</td>
			<td title=ARMv7>A5</td>
			<td colspan=48></td>
			<!-- 2011 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v5></td><td prod v5></td><td prod v5></td>
			<!-- 2012 -->
			<td prod v5></td><td prod v5></td><td prod v5></td><td prod v5></td><td prod v5></td><td prod></td><td prod></td><td prod></td><td prod v6></td><td prod v6></td><td prod v6></td><td prod v6></td>
			<!-- 2013 -->
			<td prod v6></td><td prod v6></td><td prod v6></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td>
			<!-- 2014 -->
			<td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod></td><td prod></td><td prod v8 colspan=4 data-version=8></td>
			<!-- 2015 -->
			<td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v9 colspan=4 data-version=9></td>
			<!-- 2016 -->
			<td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td>
			<!-- 2017 -->
			<td v9 colspan=12></td>
			<!-- 2018 -->
			<td v9 colspan=12></td>
			<!-- 2019 -->
			<td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td></td><td></td><td></td><td></td><td></td>
		</tr>
		<tr>
			<td>iPhone 5</td>
			<td title=ARMv7s>A6</td>
			<td colspan=60></td>
			<!-- 2012 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v6></td><td prod v6></td><td prod v6></td><td prod v6></td>
			<!-- 2013 -->
			<td prod v6></td><td prod v6></td><td prod v6></td><td prod v6></td><td prod v6></td><td prod></td><td prod></td><td prod></td><td prod v7></td><td v7></td><td v7></td><td v7></td>
			<!-- 2014 -->
			<td v7></td><td v7></td><td v7></td><td v7></td><td v7></td><td v7></td><td></td><td></td><td v8></td><td v8></td><td v8></td><td v8></td>
			<!-- 2015 -->
			<td v8></td><td v8></td><td v8></td><td v8></td><td v8></td><td v8></td><td v8></td><td v8></td><td v9></td><td v9></td><td v9></td><td v9></td>
			<!-- 2016 -->
			<td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v10 colspan=4 data-version=10></td>
			<!-- 2017 -->
			<td v10 colspan=12></td>
			<!-- 2018 -->
			<td v10 colspan=12></td>
			<!-- 2019 -->
			<td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td></td><td></td><td></td><td></td><td></td>
		</tr>
		<tr>
			<td>iPhone 5C</td>
			<td title=ARMv7>A6</td>
			<td colspan=72></td>
			<!-- 2013 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td>
			<!-- 2014 -->
			<td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod></td><td prod></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td>
			<!-- 2015 -->
			<td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td>
			<!-- 2016 -->
			<td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td>
			<!-- 2017 -->
			<td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td></td><td></td><td></td><td></td><td></td>
		</tr>
		<tr>
			<td>iPhone 5S</td>
			<td title=ARMv8>A7</td>
			<td colspan=72></td>
			<!-- 2013 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td>
			<!-- 2014 -->
			<td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod v7></td><td prod></td><td prod></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td>
			<!-- 2015 -->
			<td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td>
			<!-- 2016 -->
			<td prod v9></td><td prod v9></td><td prod v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v9></td><td v10></td><td v10></td><td v10></td><td v10></td>
			<!-- 2017 -->
			<td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td v10></td><td></td><td v11 colspan=4 data-version=11></td>
			<!-- 2018 -->
			<td v11></td><td v11></td><td v11></td><td v11></td><td v11></td><td v11></td><td v11></td><td></td><td v12 colspan=4 data-version=12></td>
			<!-- 2019 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2020 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2021 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2022 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2023 -->
			<td v12></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
		</tr>
		<tr>
			<td>iPhone 6/Plus</td>
			<td title=ARMv8>A8</td>
			<td colspan=84></td>
			<!-- 2014 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td>
			<!-- 2015 -->
			<td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v8></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td>
			<!-- 2016 -->
			<td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td>
			<!-- 2017 -->
			<td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2019 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2020 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2021 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2022 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2023 -->
			<td v12></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td>
		</tr>
		<tr unsupported>
			<td>iPhone 6S/Plus</td>
			<td title=ARMv8>A9</td>
			<td colspan=96></td>
			<!-- 2015 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td>
			<!-- 2016 -->
			<td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td>
			<!-- 2017 -->
			<td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2019 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v13 colspan=4 data-version=13></td>
			<!-- 2020 -->
			<td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13 v14></td><td v14 colspan=3 data-version=14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15 colspan=2 data-version=15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td>
			<!-- 2023 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td></td>
		</tr>
		<tr>
			<td>iPhone SE (1<sup>st</sup> gen.)</td>
			<td title=ARMv8>A9</td>
			<td colspan=108></td>
			<!-- 2016 -->
			<td></td><td></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v9></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td>
			<!-- 2017 -->
			<td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2019 -->
			<td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v12></td><td v13></td><td v13></td><td v13></td><td v13></td>
			<!-- 2020 -->
			<td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13 v14></td><td v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td>
			<!-- 2023 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td></td>
		</tr>
		<tr>
			<td>iPhone 7/Plus</td>
			<td title="ARMv8.1">A10</td>
			<td colspan=108></td>
			<!-- 2016 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td>
			<!-- 2017 -->
			<td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod v10></td><td prod></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td>
			<!-- 2019 -->
			<td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v13></td><td v13></td><td v13></td><td v13></td>
			<!-- 2020 -->
			<td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13 v14></td><td v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td>
			<!-- 2023 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td></td>
		</tr>
		<tr>
			<td>iPhone 8/Plus</td>
			<td title="ARMv8.2">A11</td>
			<td colspan=120></td>
			<!-- 2017 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td>
			<!-- 2019 -->
			<td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td>
			<!-- 2020 -->
			<td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td v13></td><td v13></td><td v13></td><td v13 ></td><td v13 v14></td><td v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v16></td>
		</tr>
		<tr>
			<td>iPhone X</td>
			<td title="ARMv8.2">A11</td>
			<td colspan=120></td>
			<!-- 2017 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v11></td><td prod v11></td>
			<!-- 2018 -->
			<td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod v11></td><td prod></td><td prod v12></td><td v12></td><td v12></td><td v12></td>
			<!-- 2019 -->
			<td  v12></td><td  v12></td><td  v12></td><td  v12></td><td  v12></td><td  v12></td><td  v12></td><td  v12></td><td  v13></td><td  v13></td><td  v13></td><td  v13></td>
			<!-- 2020 -->
			<td  v13></td><td  v13></td><td  v13></td><td  v13></td><td v13></td><td v13></td><td v13></td><td v13 ></td><td v13 v14></td><td v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v16></td>
		</tr>
		<tr discontinued>
			<td>iPhone XS/Max</td>
			<td title="ARMv8.3">A12</td>
			<td colspan=132></td>
			<!-- 2018 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td>
			<!-- 2019 -->
			<td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v13></td><td v13></td><td v13></td><td v13></td>
			<!-- 2020 -->
			<td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13></td><td v13 ></td><td v13 v14></td><td v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone XR</td>
			<td title="ARMv8.3">A12</td>
			<td colspan=132></td>
			<!-- 2018 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v12></td><td prod v12></td><td prod v12></td>
			<!-- 2019 -->
			<td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v12></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td>
			<!-- 2020 -->
			<td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13 ></td><td prod v13 v14></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 11</td>
			<td title="ARMv8.4">A13</td>
			<td colspan=144></td>
			<!-- 2019 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td>
			<!-- 2020 -->
			<td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13 v14></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td prod v14 v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 ></td><td prod v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 11 Pro/Max</td>
			<td title="ARMv8.4">A13</td>
			<td colspan=144></td>
			<!-- 2019 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td>
			<!-- 2020 -->
			<td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13 v14></td><td prod v14></td><td v14></td><td v14></td>
			<!-- 2021 -->
			<td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14></td><td v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone SE (2<sup>nd</sup> gen.)</td>
			<td title="ARMv8.4">A13</td>
			<td colspan=156></td>
			<!-- 2020 -->
			<td></td><td></td><td></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13></td><td prod v13 v14></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td prod v14 v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 12</td>
			<td title="ARMv8.5">A14</td>
			<td colspan=156></td>
			<!-- 2020 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td prod v14 v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td>
			<!-- 2023 -->
			<td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 12 Mini</td>
			<td title="ARMv8.5">A14</td>
			<td colspan=156></td>
			<!-- 2020 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td prod v14 v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 12 Pro/Max</td>
			<td title="ARMv8.5">A14</td>
			<td colspan=156></td>
			<!-- 2020 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v14></td><td prod v14></td><td prod v14></td>
			<!-- 2021 -->
			<td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14></td><td prod v14 v15></td><td v14 v15></td><td v15></td><td v15></td>
			<!-- 2022 -->
			<td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 13</td>
			<td title="ARMv8.5">A15</td>
			<td colspan=168></td>
			<!-- 2021 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td>
			<!-- 2023 -->
			<td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 13 Mini</td>
			<td title="ARMv8.5">A15</td>
			<td colspan=168></td>
			<!-- 2021 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td>
			<!-- 2023 -->
			<td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 13 Pro/Max</td>
			<td title="ARMv8.5">A15</td>
			<td colspan=168></td>
			<!-- 2021 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td>
			<!-- 2022 -->
			<td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td>
			<!-- 2023 -->
			<td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16></td><td v15 v16 v17></td><td v16 v17></td>
		</tr>
		<tr>
			<td>iPhone SE (3<sup>rd</sup> gen.)</td>
			<td title="ARMv8.5">A15</td>
			<td colspan=180></td>
			<!-- 2022 -->
			<td></td><td></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td>
			<!-- 2023 -->
			<td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16></td><td prod v15 v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 14/Plus</td>
			<td title="ARMv8.5">A15</td>
			<td colspan=180></td>
			<!-- 2022 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td>
			<!-- 2023 -->
			<td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 14 Pro/Max</td>
			<td title="ARMv8.6">A16</td>
			<td colspan=180></td>
			<!-- 2022 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td>
			<!-- 2023 -->
			<td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16></td><td prod v16 v17></td><td prod v16 v17></td>
		</tr>
		<tr>
			<td>iPhone 15/Plus</td>
			<td title="ARMv8.6">A16</td>
			<td colspan=192></td>
			<!-- 2023 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v17></td><td prod v17></td>
		</tr>
		<tr>
			<td>iPhone 15 Pro/Max</td>
			<td title="ARMv8.7">A17</td>
			<td colspan=192></td>
			<!-- 2023 -->
			<td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td prod v17></td><td prod v17></td>
		</tr>
	</tbody>
</table>

<p><small><i>A15, A16, A17, etc are names of Apple-designed systems on a chip (SoC). Each of these includes many components such as CPU, GPU, RAM, etc. The CPUs of each SoC implement some specific version of an ARM architecture (ARMv7, ARMv8.5, etc). The architecture version determines specific aspects and capabilities of the CPU, including 32- and 64-bit support (referred to as ARM32 or AArch32, and ARM64 or AArch64). When not listed, it can generally be assumed that iOS devices utilize the <b>application profile</b> variant of a given architecture version (e.g., ARMv8.5-A). Some early SoCs included Cortex CPU cores, which were not designed by Apple.</i></small></p>
