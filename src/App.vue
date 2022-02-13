<template>
	<div class="bg-blue min-h-screen p-4 lg:p-8 flex text-white font-main">
		<div class="bg-darkBlue w-full p-4 lg:p-8 flex flex-col rounded-lg">
			<header
				class="bg-blue p-4 rounded flex flex-col justify-center lg:justify-between lg:flex-row items-center gap-4 z-10"
			>
				<div class="flex items-center justify-between">
					<a href="/" class="flex justify-center lg:justify-start items-center text-3xl font-black">
						<img src="../src/assets/logo.png" alt="logo" class="w-8" />
						<span class="text-red">covid</span>
						<span>tracker</span>
					</a>
					<a href="https://static.pipezero.com/covid/data.json" target="_blank">
						<img src="../src/assets/api.png" alt="api" class="w-8 h-8 ml-4" />
					</a>
				</div>
				<div>
					<input
						class="px-4 py-2 min-w-[14rem] text-sm sm:min-w-[15rem] sm:text-base rounded text-darkBlue text-center"
						list="locations"
						placeholder="Chọn Tỉnh / Thành Phố"
						@change="handleChange($event)"
						@focus="handleFocus($event)"
					/>
					<datalist id="locations">
						<option v-for="location in getSortedLocations()" :key="location.name" :value="location.name">
							{{ location.name }}
						</option>
					</datalist>
				</div>
			</header>
			<main class="grid grid-cols-1 lg:grid-cols-2 gap-4 flex-grow mt-4 lg:mt-8">
				<section class="order-2 lg:order-1 bg-blue rounded overflow-hidden flex flex-col">
					<div class="p-4 bg-lightBlue uppercase font-black text-gray-300">Thế Giới</div>
					<div class="p-4 grid grid-cols-1 sm:grid-cols-2 gap-2 font-bold flex-grow">
						<div
							v-for="(value, title) in options"
							:key="title"
							:class="{ 'sm:col-span-2': value === 'Tổng' }"
							class="bg-lightBlue rounded overflow-hidden flex flex-col"
						>
							<div class="bg-darkBlue py-2 px-4 text-gray-400">{{ value }}</div>
							<div class="text-center p-4 text-xl text-gray-300 flex flex-col items-center justify-center flex-grow">
								{{ getWorldData(title) }}
								<span v-if="!getWorldData(title)" class="text-xs text-gray-400 font-thin italic"
									>(hoặc chưa được cập nhật)</span
								>
							</div>
						</div>
					</div>
				</section>
				<section class="order-2 lg:order-1 bg-blue rounded overflow-hidden flex flex-col">
					<div class="p-4 bg-lightBlue uppercase font-black text-gray-300">Việt Nam</div>
					<div class="p-4 grid grid-cols-1 sm:grid-cols-2 gap-2 font-bold flex-grow">
						<div
							v-for="(value, title) in options"
							:key="title"
							:class="{ 'sm:col-span-2': value === 'Tổng' }"
							class="bg-lightBlue rounded overflow-hidden flex flex-col"
						>
							<div class="bg-darkBlue py-2 px-4 text-gray-400">{{ value }}</div>
							<div class="text-center p-4 text-xl text-gray-300 flex flex-col items-center justify-center flex-grow">
								{{ getInternalData(title) }}
								<span v-if="!getInternalData(title)" class="text-xs text-gray-400 font-thin italic"
									>(hoặc chưa được cập nhật)</span
								>
							</div>
						</div>
					</div>
				</section>
				<section class="order-2 lg:order-1 bg-lightBlue p-4 rounded flex justify-center items-center">
					<LineChart :chartData="chartData" :options="chartOptions" />
				</section>
				<section
					class="order-1 lg:order-2 bg-blue rounded overflow-hidden flex-col lg:flex"
					:class="{ hidden: !selectedLocation }"
				>
					<div class="p-4 bg-lightBlue uppercase font-black text-gray-300">
						{{ selectedLocation || "Tỉnh / Thành Phố" }}
					</div>
					<div class="p-4 grid grid-cols-1 sm:grid-cols-2 gap-2 font-bold">
						<div
							v-for="(value, title) in options"
							:key="title"
							:class="{ 'sm:col-span-2': value === 'Tổng' }"
							class="bg-lightBlue rounded overflow-hidden flex flex-col"
						>
							<div class="bg-darkBlue py-2 px-4 text-gray-400">{{ value }}</div>
							<div class="text-center p-4 text-xl text-gray-300 flex flex-col items-center justify-center flex-grow">
								{{ getSelectedLocationData(title) }}
								<span v-if="!getSelectedLocationData(title)" class="text-xs text-gray-400 font-thin italic"
									>(hoặc chưa được cập nhật)</span
								>
							</div>
						</div>
					</div>
				</section>
			</main>
		</div>
	</div>
</template>

<script>
import { Chart, registerables } from "chart.js"
import { LineChart } from "vue-chart-3"

Chart.register(...registerables)

export default {
	name: "App",
	components: {
		LineChart,
	},
	data() {
		return {
			api: "https://static.pipezero.com/covid/data.json",
			chartData: {
				labels: [],
				datasets: [
					{
						data: [],
						borderColor: "#d1d5db",
						backgroundColor: "#d1d5db",
						label: "Ca nhiễm mới trong ngày tại Việt Nam",
					},
				],
			},
			chartOptions: {
				scales: {
					x: {
						display: true,
						title: {
							display: true,
							text: "Ngày",
							color: "#d1d5db",
							font: {
								size: 16,
							},
						},
						ticks: {
							color: "#d1d5db",
							font: {
								size: 10,
							},
						},
					},
					y: {
						display: true,
						title: {
							display: true,
							text: "Số ca",
							color: "#d1d5db",
							font: {
								size: 16,
							},
						},
						ticks: {
							color: "#d1d5db",
							font: {
								size: 10,
							},
						},
					},
				},
				plugins: {},
			},
			statistics: {
				world: null,
				internal: null,
				locations: null,
			},
			selectedLocation: null,
			options: {
				casesToday: "Hôm nay",
				treating: "Đang điều trị",
				recovered: "Hồi phục",
				death: "Tử vong",
				cases: "Tổng",
			},
		}
	},
	mounted() {
		fetch(this.api)
			.then((res) => res.json())
			.then((data) => {
				this.statistics.world = data.total.world
				this.statistics.world.casesToday = data.today.world.cases
				this.statistics.internal = data.total.internal
				this.statistics.internal.casesToday = data.today.internal.cases
				this.statistics.locations = data.locations
				this.chartData.labels = data.overview.map((item) => item.date)
				this.chartData.datasets[0].data = data.overview.map((item) => item.cases)
			})
	},
	methods: {
		getSortedLocations() {
			if (this.statistics.locations) {
				return this.statistics.locations.sort((a, b) => new Intl.Collator("vi").compare(a.name, b.name))
			}
			return "Không có dữ liệu!"
		},

		getSelectedLocationData(key) {
			if (this.selectedLocation) {
				return this.statistics.locations.find((location) => location.name === this.selectedLocation)[key]
			}
			return 0
		},

		getWorldData(key) {
			if (this.statistics.world) {
				return this.statistics.world[key]
			}
			return 0
		},

		getInternalData(key) {
			if (this.statistics.internal) {
				return this.statistics.internal[key]
			}
			return 0
		},

		handleChange(event) {
			this.selectedLocation = event.target.value
			event.target.blur()
		},

		handleFocus(event) {
			event.target.value = null
		},
	},
}
</script>
