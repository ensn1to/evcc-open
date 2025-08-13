<template>
	<div class="power-chart">
		<canvas ref="chartCanvas" :width="chartWidth" :height="chartHeight"></canvas>
	</div>
</template>

<script>
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);

export default {
	name: "PowerChart",
	props: {
		data: {
			type: Array,
			default: () => [],
		},
		dataType: {
			type: String,
			default: "sitepower",
		},
		selectedDate: {
			type: String,
			default: "",
		},
	},
	data() {
		return {
			chart: null,
			chartWidth: 800,
			chartHeight: 400,
		};
	},
	computed: {
		chartData() {
			if (!this.data || this.data.length === 0) {
				return {
					labels: [],
					datasets: [{
						label: this.dataType === 'sitepower' ? 'Site Power (kW)' : 'Battery Power (kW)',
						data: [],
						borderColor: this.dataType === 'sitepower' ? '#007bff' : '#28a745',
						backgroundColor: this.dataType === 'sitepower' ? 'rgba(0, 123, 255, 0.1)' : 'rgba(40, 167, 69, 0.1)',
						borderWidth: 2,
						fill: true,
						tension: 0.4,
						pointRadius: 3,
						pointHoverRadius: 5,
						pointBackgroundColor: this.dataType === 'sitepower' ? '#007bff' : '#28a745',
						pointBorderColor: '#fff',
						pointBorderWidth: 2,
					}],
				};
			}

			return {
				labels: this.data.map(point => {
					const date = new Date(point.time);
					return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
				}),
				datasets: [{
					label: this.dataType === 'sitepower' ? 'Site Power (kW)' : 'Battery Power (kW)',
					data: this.data.map(point => point.power || 0),
					borderColor: this.dataType === 'sitepower' ? '#007bff' : '#28a745',
					backgroundColor: this.dataType === 'sitepower' ? 'rgba(0, 123, 255, 0.1)' : 'rgba(40, 167, 69, 0.1)',
					borderWidth: 2,
					fill: false, // 禁用填充避免filler插件错误
					tension: 0.4,
					pointRadius: 3,
					pointHoverRadius: 5,
					pointBackgroundColor: this.dataType === 'sitepower' ? '#007bff' : '#28a745',
					pointBorderColor: '#fff',
					pointBorderWidth: 2,
				}],
			};
		},
		chartOptions() {
			return {
				responsive: true,
				maintainAspectRatio: false,
				animation: false, // 禁用动画避免渲染问题
				scales: {
					x: {
						title: {
							display: true,
							text: 'Time',
							color: '#666',
							font: {
								size: 12,
								weight: 'normal'
							}
						},
						grid: {
							display: true,
							color: 'rgba(0, 0, 0, 0.1)',
							drawBorder: false,
						},
						ticks: {
							color: '#666',
							font: {
								size: 11
							}
						}
					},
					y: {
						title: {
							display: true,
							text: 'Power (kW)',
							color: '#666',
							font: {
								size: 12,
								weight: 'normal'
							}
						},
						grid: {
							display: true,
							color: 'rgba(0, 0, 0, 0.1)',
							drawBorder: false,
						},
						ticks: {
							color: '#666',
							font: {
								size: 11
							}
						},
						beginAtZero: false,
					},
				},
				plugins: {
					legend: {
						display: true,
						position: 'top',
						align: 'start',
						labels: {
							usePointStyle: true,
							padding: 20,
							font: {
								size: 12
							}
						}
					},
					tooltip: {
						mode: 'index',
						intersect: false,
						backgroundColor: 'rgba(0, 0, 0, 0.8)',
						titleColor: '#fff',
						bodyColor: '#fff',
						borderColor: 'rgba(0, 0, 0, 0.1)',
						borderWidth: 1,
						callbacks: {
							label: (context) => {
								return `${context.dataset.label}: ${context.parsed.y.toFixed(2)} kW`;
							},
						},
					},
				},
				interaction: {
					mode: 'nearest',
					axis: 'x',
					intersect: false,
				},
				elements: {
					line: {
						tension: 0.4
					}
				}
			};
		},
	},
	mounted() {
		this.initChart();
		this.handleResize();
		window.addEventListener('resize', this.handleResize);
	},
	beforeUnmount() {
		if (this.chart) {
			this.chart.destroy();
		}
		window.removeEventListener('resize', this.handleResize);
	},
	watch: {
		data: {
			handler() {
				this.updateChart();
			},
			deep: true,
		},
		dataType() {
			this.updateChart();
		},
	},
	methods: {
		initChart() {
			try {
				const ctx = this.$refs.chartCanvas.getContext('2d');
				if (this.chart) {
					this.chart.destroy();
				}
				this.chart = new Chart(ctx, {
					type: 'line',
					data: this.chartData,
					options: this.chartOptions,
				});
			} catch (error) {
				console.error('Error initializing chart:', error);
			}
		},
		updateChart() {
			if (this.chart) {
				try {
					this.chart.data = this.chartData;
					this.chart.update('none'); // Use 'none' animation mode for better performance
				} catch (error) {
					console.error('Error updating chart:', error);
					// Reinitialize chart if update fails
					this.chart.destroy();
					this.initChart();
				}
			}
		},
		handleResize() {
			const container = this.$el;
			if (container) {
				this.chartWidth = container.clientWidth;
				this.chartHeight = Math.min(400, container.clientWidth * 0.5);
			}
		},
	},
};
</script>

<style scoped>
.power-chart {
	position: relative;
	width: 100%;
	height: 400px;
	margin: 1rem 0;
}

canvas {
	max-width: 100%;
	height: auto;
}
</style>