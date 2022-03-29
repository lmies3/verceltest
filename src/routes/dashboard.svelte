<script>
	import { onMount } from 'svelte';
	import { supabase } from '../supabaseclient';
	import {
		DatePicker,
		DatePickerInput,
		DataTable,
		Toolbar,
		ToolbarContent,
		ToolbarSearch,
		DataTableSkeleton,
		Tabs,
		Tab,
		TabContent,
		Select,
		SelectItem, 
		Button,
		ButtonSet,
		Grid,
		Row,
		Column
	} from 'carbon-components-svelte';
	import * as Pancake from '@sveltejs/pancake';

	let searchdate = '2022-01-05';
	let n = 100000;

	let summaries = [];
	async function getSummaries() {
		comments = [];
		summaries = [];
		let { data, error } = await supabase.rpc('get_summaries', {
			searchdate
		});
		if (error) console.error(error);
		else
			data.forEach((item, i) => {
				item.id = String(i + 1);
			});

		summaries = data;
		getComments();
	}

	let comments = [];
	async function getComments() {
		let { data, error } = await supabase.rpc('get_comments_fordate', {
			searchdate,
			n
		});
		if (error) console.error(error);
		else comments = data;
	}

	let startdate = '2022-01-05';
	let enddate = '2022-01-19';
	let tickers = [];
	let selected;
	async function getTickers() {
		let { data, error } = await supabase.rpc('get_tickers_list', {
			enddate,
			startdate
		});
		if (error) console.error(error);
		else tickers = data;
	}

	let closest;
	let searchticker;
	let closepoints = [];
	let openpoints = [];
	let highpoints = [];
	let lowpoints = [];

	let close = [];
	let open = [];
	let low = [];
	let high = [];

	let x1 = +Infinity;
	let x2 = -Infinity;
	let y1 = +Infinity;
	let y2 = -Infinity;



	function clickChartDate() {
		searchdate = closest.x.toString().slice(0, 4) + "-" + closest.x.toString().slice(4, 6) + "-" + closest.x.toString().slice(6);
		getSummaries();
	}

	async function getChartData() {

		closepoints = [];
		openpoints = [];
		highpoints = [];
		lowpoints = [];
		close = [];
		open = [];
		low = [];
		high = [];

		searchticker = selected;

		let { data, error } = await supabase.rpc('get_ticker_actual', {
			searchticker,
			startdate,
			enddate
		});

		if (error) console.error(error);
		else 
		closepoints.push({ name: searchticker, data: [] });
		openpoints.push({ name: searchticker, data: [] });
		highpoints.push({ name: searchticker, data: [] });
		lowpoints.push({ name: searchticker, data: [] });


		data.forEach((item, i) => {
			let x = parseInt(item.date.replaceAll('-', ''));
			closepoints[0].data.push({ x: x, y: item.close });
			openpoints[0].data.push({ x: x, y: item.open });
			highpoints[0].data.push({ x: x, y: item.high });
			lowpoints[0].data.push({ x: x, y: item.low });

		});

		x1 = +Infinity;
		x2 = -Infinity;
		y1 = +Infinity;
		y2 = -Infinity;

		closepoints.forEach((ticker) => {
			ticker.data.forEach((d) => {
				if (d.x < x1) x1 = d.x;
				if (d.x > x2) x2 = d.x;
				if (d.y < y1) y1 = d.y;
				if (d.y > y2) y2 = d.y;
			});
		});

		openpoints.forEach((ticker) => {
			ticker.data.forEach((d) => {
				if (d.x < x1) x1 = d.x;
				if (d.x > x2) x2 = d.x;
				if (d.y < y1) y1 = d.y;
				if (d.y > y2) y2 = d.y;
			});
		});

		highpoints.forEach((ticker) => {
			ticker.data.forEach((d) => {
				if (d.x < x1) x1 = d.x;
				if (d.x > x2) x2 = d.x;
				if (d.y < y1) y1 = d.y;
				if (d.y > y2) y2 = d.y;
			});
		});

		lowpoints.forEach((ticker) => {
			ticker.data.forEach((d) => {
				if (d.x < x1) x1 = d.x;
				if (d.x > x2) x2 = d.x;
				if (d.y < y1) y1 = d.y;
				if (d.y > y2) y2 = d.y;
			});
		});

		close = closepoints.reduce((close, ticker) => {
			return close.concat(
				ticker.data.map((d) => ({
					x: d.x,
					y: d.y,
					ticker
				}))
			);
		}, []);

		open = openpoints.reduce((open, ticker) => {
			return open.concat(
				ticker.data.map((d) => ({
					x: d.x,
					y: d.y,
					ticker
				}))
			);
		}, []);

		low = lowpoints.reduce((low, ticker) => {
			return low.concat(
				ticker.data.map((d) => ({
					x: d.x,
					y: d.y,
					ticker
				}))
			);
		}, []);

		high = highpoints.reduce((high, ticker) => {
			return high.concat(
				ticker.data.map((d) => ({
					x: d.x,
					y: d.y,
					ticker
				}))
			);
		}, []);

	}

	onMount(async () => {
		getSummaries();
		selected = 'TSLA';
		getTickers();
		getChartData();
	});

</script>

<svelte:head>
	<title></title>
</svelte:head>

<!-- MAIN GRID -->
<Grid narrow>
	<!-- PRICING HEADER -->
	<Row>
		<Column>
			<h2>Charts</h2>
			<hr />
		</Column>
	</Row>
	<!-- PRICING SELECTIONS -->
	<Row style="margin-top:14px">
		<Column lg={4}>
			<DatePicker
				dateFormat="Y-m-d"
				datePickerType="range"
				bind:valueFrom={startdate}
				bind:valueTo={enddate}
				on:change
			>
				<DatePickerInput labelText="Start date" />
				<DatePickerInput labelText="End date" />
			</DatePicker>
			<Button size="small" kind="ghost" on:click={getTickers}>Update Tickers</Button>
		</Column>
		<Column lg={2}>
			<Select labelText="Ticker ({tickers.length})" bind:selected>
				{#each tickers as ticker}
					<SelectItem value={ticker.ticker} text={ticker.ticker} />
				{/each}
			</Select>
			<Button size="small" kind="ghost" on:click={getChartData}>Get Charts</Button>
		</Column>
	</Row>
	<!-- PRICING CHART -->
	<Row style="margin-top:14px">
		<Column>
			<Tabs>
				<Tab label="Close" />
				<Tab label="Open" />
				<Tab label="High" />
				<Tab label="Low" />
				<svelte:fragment slot="content">
					<TabContent>
						<div class="chart">
							<Pancake.Chart {x1} {x2} {y1} {y2}>
								<Pancake.Svg>
									{#if close.length > 0}
										<Pancake.SvgLine data={close} let:d>
											<path class="data" {d} />
										</Pancake.SvgLine>
									{/if}
									{#if closest}
										<Pancake.SvgLine data={closest.ticker.data} let:d>
											<path class="highlight" {d} />
										</Pancake.SvgLine>
									{/if}
								</Pancake.Svg>
								<Pancake.Grid horizontal count={5} let:value>
									<div class="grid-line horizontal"><span>${value}</span></div>
								</Pancake.Grid>
								<Pancake.Grid vertical count={5} let:value>
									<span class="x-label">{value.toString().slice(4, 6) + "-" + value.toString().slice(6)}</span>
								</Pancake.Grid>
								{#if closest}
									<Pancake.Point x={closest.x} y={closest.y}>
										<span class="annotation-point" />
										<div
											class="annotation"
											style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)"
										>
											<strong>{searchticker}</strong>
											<a href="" on:click={clickChartDate}>{closest.x.toString().slice(0, 4) + "-" + closest.x.toString().slice(4, 6) + "-" + closest.x.toString().slice(6)}</a>
											<span on:click={clickChartDate}>${closest.y.toFixed(2)}</span>
										</div>
									</Pancake.Point>
								{/if}
								<Pancake.Quadtree data={close} bind:closest />
							</Pancake.Chart>
						</div>
					</TabContent>
					<TabContent>
						<div class="chart">
							<Pancake.Chart {x1} {x2} {y1} {y2}>
								<Pancake.Svg>
									{#if open.length > 0}
										<Pancake.SvgLine data={open} let:d>
											<path class="data" {d} />
										</Pancake.SvgLine>
									{/if}
									{#if closest}
										<Pancake.SvgLine data={closest.ticker.data} let:d>
											<path class="highlight" {d} />
										</Pancake.SvgLine>
									{/if}
								</Pancake.Svg>
								<Pancake.Grid horizontal count={5} let:value>
									<div class="grid-line horizontal"><span>${value}</span></div>
								</Pancake.Grid>
								<Pancake.Grid vertical count={5} let:value>
									<span class="x-label">{value.toString().slice(4, 6) + "-" + value.toString().slice(6)}</span>
								</Pancake.Grid>
								{#if closest}
									<Pancake.Point x={closest.x} y={closest.y}>
										<span class="annotation-point" />
										<div
											class="annotation"
											style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)"
										>
											<strong>{searchticker}</strong>
											<a href="" on:click={clickChartDate}>{closest.x.toString().slice(0, 4) + "-" + closest.x.toString().slice(4, 6) + "-" + closest.x.toString().slice(6)}</a>
											<span on:click={clickChartDate}>${closest.y.toFixed(2)}</span>
										</div>
									</Pancake.Point>
								{/if}
								<Pancake.Quadtree data={open} bind:closest />
							</Pancake.Chart>
						</div>
					</TabContent>
					<TabContent>
						<div class="chart">
							<Pancake.Chart {x1} {x2} {y1} {y2}>
								<Pancake.Svg>
									{#if high.length > 0}
										<Pancake.SvgLine data={high} let:d>
											<path class="data" {d} />
										</Pancake.SvgLine>
									{/if}
									{#if closest}
										<Pancake.SvgLine data={closest.ticker.data} let:d>
											<path class="highlight" {d} />
										</Pancake.SvgLine>
									{/if}
								</Pancake.Svg>
								<Pancake.Grid horizontal count={5} let:value>
									<div class="grid-line horizontal"><span>${value}</span></div>
								</Pancake.Grid>
								<Pancake.Grid vertical count={5} let:value>
									<span class="x-label">{value.toString().slice(4, 6) + "-" + value.toString().slice(6)}</span>
								</Pancake.Grid>
								{#if closest}
									<Pancake.Point x={closest.x} y={closest.y}>
										<span class="annotation-point" />
										<div
											class="annotation"
											style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)"
										>
											<strong>{searchticker}</strong>
											<a href="" on:click={clickChartDate}>{closest.x.toString().slice(0, 4) + "-" + closest.x.toString().slice(4, 6) + "-" + closest.x.toString().slice(6)}</a>
											<span on:click={clickChartDate}>${closest.y.toFixed(2)}</span>
										</div>
									</Pancake.Point>
								{/if}
								<Pancake.Quadtree data={high} bind:closest />
							</Pancake.Chart>
						</div>
					</TabContent>
					<TabContent>
						<div class="chart">
							<Pancake.Chart {x1} {x2} {y1} {y2}>
								<Pancake.Svg>
									{#if low.length > 0}
										<Pancake.SvgLine data={low} let:d>
											<path class="data" {d} />
										</Pancake.SvgLine>
									{/if}
									{#if closest}
										<Pancake.SvgLine data={closest.ticker.data} let:d>
											<path class="highlight" {d} />
										</Pancake.SvgLine>
									{/if}
								</Pancake.Svg>
								<Pancake.Grid horizontal count={5} let:value>
									<div class="grid-line horizontal"><span>${value}</span></div>
								</Pancake.Grid>
								<Pancake.Grid vertical count={5} let:value>
									<span class="x-label">{value.toString().slice(4, 6) + "-" + value.toString().slice(6)}</span>
								</Pancake.Grid>
								{#if closest}
									<Pancake.Point x={closest.x} y={closest.y}>
										<span class="annotation-point" />
										<div
											class="annotation"
											style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)"
										>
											<strong>{searchticker}</strong>
											<a href="" on:click={clickChartDate}>{closest.x.toString().slice(0, 4) + "-" + closest.x.toString().slice(4, 6) + "-" + closest.x.toString().slice(6)}</a>
											<span on:click={clickChartDate}>${closest.y.toFixed(2)}</span>
										</div>
									</Pancake.Point>
								{/if}
								<Pancake.Quadtree data={low} bind:closest />
							</Pancake.Chart>
						</div>
					</TabContent>

				</svelte:fragment>
				</Tabs>
			<!-- <input placeholder="Type to filter" bind:value={filter} /> -->
			
			<style>
				.chart {
					height: 400px;
					padding: 3em 0 2em 2em;
					margin: 0 0 36px 0;
				}

				input {
					font-size: inherit;
					font-family: inherit;
					padding: 0.5em;
				}

				.grid-line {
					position: relative;
					display: block;
				}

				.grid-line.horizontal {
					width: calc(100% + 2em);
					left: -2em;
					border-bottom: 1px dashed #ccc;
				}

				.grid-line span {
					position: absolute;
					left: 0;
					bottom: 2px;
					font-family: sans-serif;
					font-size: 14px;
					color: #999;
				}

				.x-label {
					position: absolute;
					width: 4em;
					left: -2em;
					bottom: -22px;
					font-family: sans-serif;
					font-size: 14px;
					color: #999;
					text-align: center;
				}

				path.data {
					stroke: rgba(0, 0, 0, 0.3);
					stroke-linejoin: round;
					stroke-linecap: round;
					stroke-width: 1px;
					fill: none;
				}

				.highlight {
					stroke: #5C62FE;
					fill: none;
					stroke-width: 2;
				}
				.annotation {
					position: absolute;
					white-space: nowrap;
					bottom: 1em;
					line-height: 1.2;
					background-color: rgba(255, 255, 255, 0.9);
					padding: 0.2em 0.4em;
					border-radius: 2px;
				}

				.annotation-point {
					position: absolute;
					width: 10px;
					height: 10px;
					background-color: #5C62FE;
					border-radius: 50%;
					transform: translate(-50%, -50%);
				}

				.annotation strong {
					display: block;
					font-size: 20px;
				}

				.annotation span {
					display: block;
					font-size: 14px;
				}
			</style>
		</Column>
	</Row>
	<!-- COMMENTS HEADER -->
	<Row style="margin-top:14px">
		<Column>
			<h2>Comments</h2>
			<hr />
		</Column>
	</Row>
	<!-- COMMENTS SELECTIONS -->
	<Row style="margin-top:14px">
		<Column>
			<ButtonSet stacked>
				<DatePicker datePickerType="single" dateFormat="Y-m-d" bind:value={searchdate}>
					<DatePickerInput labelText="Comment Explore Date" />
				</DatePicker>
				<Button size="small" kind="ghost" on:click={getSummaries}>Get Comments</Button>
			</ButtonSet>
		</Column>
	</Row>
	<!-- COMMENTS TABLES -->
	<Row style="margin-top:14px">
		<Column>
			<Tabs>
				<Tab label="Ticker Summaries" />
				<Tab label="All Comments" />
				<svelte:fragment slot="content">
					<TabContent
						>{#if comments.length > 0}
							<DataTable
								sortable
								zebra
								headers={[
									{ key: 'thread_date', value: 'date', sort: false },
									{ key: 'ticker', value: 'ticker' },
									{ key: 'total_count', value: 'total' },
									{ key: 'positive_count', value: 'positive' },
									{ key: 'neutral_count', value: 'neutral' },
									{ key: 'negative_count', value: 'negative' },
									{ key: 'top_comment_score', value: 'top score' }
								]}
								rows={summaries}
							>
								<Toolbar>
									<ToolbarContent>
										<ToolbarSearch
											persistent
											value=""
											shouldFilterRows={(row, value) => {
												return row.ticker.toLowerCase().includes(value.toLowerCase());
											}}
										/>
									</ToolbarContent>
								</Toolbar>
							</DataTable>
						{:else}
							<DataTableSkeleton
								showHeader={false}
								showToolbar={true}
								zebra
								headers={[
									{ value: 'date' },
									{ value: 'ticker' },
									{ value: 'total' },
									{ value: 'positive' },
									{ value: 'neutral' },
									{ value: 'negative' },
									{ value: 'top score' }
								]}
								rows={30}
							/>
						{/if}</TabContent
					>
					<TabContent
						>{#if comments.length > 0}
							<DataTable
								sortable
								zebra
								headers={[
									{ key: 'ticker', value: 'ticker' },
									{ key: 'author', value: 'author', sort: false },
									{ key: 'comment', value: 'comment', sort: false },
									{ key: 'score', value: 'score' }
								]}
								rows={comments}
							>
								<Toolbar>
									<ToolbarContent>
										<ToolbarSearch persistent value="" shouldFilterRows />
									</ToolbarContent>
								</Toolbar>
							</DataTable>
						{:else}
							<DataTableSkeleton
								showHeader={false}
								showToolbar={true}
								headers={[
									{ value: 'ticker' },
									{ value: 'author' },
									{ value: 'comment' },
									{ value: 'score' }
								]}
								rows={30}
							/>
						{/if}</TabContent
					>
				</svelte:fragment>
			</Tabs>
			<div style="margin-bottom:20px" />
		</Column>
	</Row>
</Grid>
