<template>
	<div id="app">
		<div class="container grid-lg">
			<div class="columns">
				<!-- title -->
				<div class="column col-4 col-md-12">
					<h1>Thunderbird<br />Email Stats</h1>
				</div>
				<!-- featured figures -->
				<div class="column col-2 col-md-3 col-sm-6 text-center">
					<div class="text-gray">Total mails</div>
					<div class="figure">{{ figure.total }}</div>
					<div class="text-gray">within {{ figure.years }} years</div>
				</div>
				<div class="column col-2 col-md-3 col-sm-6 text-center">
					<div class="text-primary">Mails received</div>
					<div class="figure text-primary">{{ figure.in }}</div>
					<div class="text-gray">{{ figure.inPercentage }}% of total</div>
				</div>
				<div class="column col-2 col-md-3 col-sm-6 text-center">
					<div class="text-secondary">Mails sent</div>
					<div class="figure text-secondary">{{ figure.out }}</div>
					<div class="text-gray">{{ figure.outPercentage }}% of total</div>
				</div>
				<div class="column col-2 col-md-3 col-sm-6 text-center">
					<div class="text-gray">Mails per day</div>
					<div class="figure">{{ figure.perday }}</div>
					<div class="text-gray">{{ figure.perweek }} mails per week</div>
				</div>
			</div>
			<!-- line charts for mail amount per year and month -->
			<div class="columns">
				<div class="column col-6 col-sm-12">
					<LineChart
						title="Years"
						description="Total number of emails per year"
						:datasets="mailsPerYear.datasets"
						:labels="mailsPerYear.labels"
					/>
				</div>
				<div class="column col-6 col-sm-12">
					<LineChart
						title="Months"
						description="Total number of emails per month"
						:datasets="mailsPerMonth.datasets"
						:labels="mailsPerMonth.labels"
					/>
				</div>
			</div>
			<!-- bar charts for mail distribution over daytime and weekday -->
			<div class="columns">
				<div class="column col-6 col-sm-12">
					<BarChart
						title="Daytime"
						description="Number of emails per time of day"
						:datasets="mailsPerHour.datasets"
						:labels="mailsPerHour.labels"
					/>
				</div>
				<div class="column col-6 col-sm-12">
					<BarChart
						title="Weekdays"
						description="Number of emails per day of week"
						:datasets="mailsPerWeekday.datasets"
						:labels="mailsPerWeekday.labels"
					/>
				</div>
			</div>
			<!-- heat map for mail distribution over daytime on weekday -->
			<div class="columns">
				<div class="column col-6 col-sm-12 heatmap">
					<HeatMap
						title="Daytime Incoming"
						description="Number of incoming emails per weekday per hour"
						rgb="48, 206, 241"
						:dataset="$options.stats.mailsPerWeekdayPerHour.in"
					/>
				</div>
				<div class="column col-6 col-sm-12 heatmap">
					<HeatMap
						title="Daytime Outgoing"
						description="Number of outgoing emails per weekday per hour"
						rgb="237, 47, 71"
						:dataset="$options.stats.mailsPerWeekdayPerHour.out"
					/>
				</div>
			</div>
			<!-- footer -->
			<div class="columns">
				<div class="column col-12 text-gray text-center text-sm">
					<p>
						This data was retrieved on {{ figure.tstamp }}<br />
						Thunderbird Stats v{{ $options.version }} - star and fork this project on <a href="https://github.com/devmount/thunderbird-stats">Github</a>
					</p>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
// internal components
import LineChart from './components/LineChart.vue'
import BarChart from './components/BarChart.vue'
import HeatMap from './components/HeatMap.vue'

// chart data
import META from './data/meta.json'
import MAILS_PER_HOUR from './data/mails-per-hour.json'
import MAILS_PER_MONTH from './data/mails-per-month.json'
import MAILS_PER_YEAR from './data/mails-per-year.json'
import MAILS_PER_WEEKDAY from './data/mails-per-weekday.json'
import MAILS_PER_WEEKDAY_PER_HOUR from './data/mails-per-weekday-per-hour.json'

// initialize Chart.js with global configuration
import Chart from 'chart.js'
Chart.defaults.global.defaultFontColor = "#7e8d97"
Chart.defaults.global.elements.arc.borderWidth = 0
Chart.defaults.global.legend.display = false
Chart.defaults.global.tooltips.mode = 'index'
Chart.defaults.global.tooltips.intersect = false
Chart.defaults.global.tooltips.multiKeyBackground = '#000'
Chart.defaults.global.tooltips.titleMarginBottom = 10
Chart.defaults.global.tooltips.xPadding = 10
Chart.defaults.global.tooltips.yPadding = 10
Chart.defaults.global.tooltips.cornerRadius = 2
Chart.defaults.global.hover.mode = 'index'

export default {
	name: 'app',
	version: '0.1.3',
	components: {
		LineChart,
		BarChart,
		HeatMap,
	},
	stats: {
		meta: META,
		mailsPerHour: MAILS_PER_HOUR,
		mailsPerMonth: MAILS_PER_MONTH,
		mailsPerYear: MAILS_PER_YEAR,
		mailsPerWeekday: MAILS_PER_WEEKDAY,
		mailsPerWeekdayPerHour: MAILS_PER_WEEKDAY_PER_HOUR,
	},
	computed: {
		figure () {
			var meta = this.$options.stats.meta
			var tstamp = new Date(meta.tstamp)
			return {
				in: meta.in.toLocaleString(),
				inPercentage: (meta.in/meta.total*100).toFixed(2),
				out: meta.out.toLocaleString(),
				outPercentage: (meta.out/meta.total*100).toFixed(2),
				total: meta.total.toLocaleString(),
				years: meta.years.toFixed(1),
				perday: (meta.total/meta.days.toFixed(1)).toFixed(2),
				perweek: (meta.total/meta.weeks.toFixed(1)).toFixed(1),
				oldest: new Date(meta.oldest),
				newest: new Date(meta.newest),
				tstamp: tstamp.toLocaleString(),
			}
		},
		mailsPerYear () {
			var din = this.$options.stats.mailsPerYear.in
			var dout = this.$options.stats.mailsPerYear.out
			var years = [], dsin = [], dsout = []
			for (let y = this.figure.oldest.getFullYear(); y <= this.figure.newest.getFullYear(); ++y) {
				years.push(y)
				dsin.push(din.hasOwnProperty(y) ? din[y] : 0)
				dsout.push(dout.hasOwnProperty(y) ? dout[y] : 0)
			}
			return {
				datasets: [
					{ label: 'Mails sent', data: dsout, color: 'rgb(237, 47, 71)', bcolor: 'rgb(237, 47, 71, .2)'  },
					{ label: 'Mails received', data: dsin,  color: 'rgb(48, 206, 242)', bcolor: 'rgb(48, 206, 242, .2)' },
				],
				labels: years
			}
		},
		mailsPerMonth () {
			var din = this.$options.stats.mailsPerMonth.in
			var dout = this.$options.stats.mailsPerMonth.out
			var months = [], dsin = [], dsout = [], skip = true
			var labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
			for (const year in din) {
				for (const month in din[year]) {
					// trim leading zero months
					if (din[year][month] == 0) {
						if (skip) continue
					} else {
						skip = false
					}
					months.push(year + ' ' + labels[month-1])
					dsin.push(din[year][month])
					if(dout.hasOwnProperty(year) && dout[year].hasOwnProperty(month)) {
						dsout.push(dout[year][month])
					} else {
						dsout.push(0)
					}
				}
			}
			return {
				datasets: [
					{ label: 'Mails sent', data: dsout, color: 'rgb(237, 47, 71)', bcolor: 'rgb(237, 47, 71, .2)'  },
					{ label: 'Mails received', data: dsin,  color: 'rgb(48, 206, 242)', bcolor: 'rgb(48, 206, 242, .2)' },
				],
				labels: months
			}
		},
		mailsPerHour () {
			var din = this.$options.stats.mailsPerHour.in
			var dout = this.$options.stats.mailsPerHour.out
			var hours = [], dsin = [], dsout = []
			for (const hour in din) {
				hours.push(hour)
				dsin.push(din[hour])
				dsout.push(dout[hour])
			}
			return {
				datasets: [
					{ label: 'Mails sent', data: dsout, color: 'rgb(237, 47, 71)', bcolor: 'rgb(237, 47, 71, .2)'  },
					{ label: 'Mails received', data: dsin,  color: 'rgb(48, 206, 242)', bcolor: 'rgb(48, 206, 242, .2)' },
				],
				labels: hours
			}
		},
		mailsPerWeekday () {
			var din = this.$options.stats.mailsPerWeekday.in
			var dout = this.$options.stats.mailsPerWeekday.out
			var weekdays = [], dsin = [], dsout = []
			var days = ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']
			for (const weekday in din) {
				weekdays.push(days[weekday])
				dsin.push(din[weekday])
				dsout.push(dout[weekday])
			}
			return {
				datasets: [
					{ label: 'Mails sent', data: dsout, color: 'rgb(237, 47, 71)', bcolor: 'rgb(237, 47, 71, .2)'  },
					{ label: 'Mails received', data: dsin,  color: 'rgb(48, 206, 242)', bcolor: 'rgb(48, 206, 242, .2)' },
				],
				labels: weekdays
			}
		},
	},
}
</script>

<style lang="scss">
// spectre config
$primary-color: #30cef1;
$secondary-color: #ed2f47;
$body-font-color: #cedae2;
$bg-color: #0d1219;
$bg-color-dark: #222627;
$bg-color-light: #0d1219;
$border-color: #222627;
$dark-color: #222627;
$gray-color: #7e8d97;
@import "node_modules/spectre.css/src/spectre";

// import fonts
@import url('https://fonts.googleapis.com/css?family=Fira+Mono');
</style>

<style lang="stylus">
font-mono = 'Fira Mono', 'Courier New', Courier, monospace

#app
	-webkit-font-smoothing antialiased
	-moz-osx-font-smoothing grayscale

.columns
	margin-top 1rem

h1, h2, h3
	text-align center
	font-weight 100

.figure
	font-size 2.75em;
	line-height 1em;
	font-weight 500;

.text-sm
	font-size .75em
.text-xs
	font-size .7em
.text-mono
	font-family font-mono

.mt-1
	margin-top .5em
.mb-1
	margin-bottom .5em
.mb-2
	margin-bottom 1em
.mr-1
	margin-right .5em

.chart
	h3
		margin-bottom .25em
	p
		margin 0

.tooltip
	position relative
	&::after
		background rgba(0, 0, 0, .8)
</style>
