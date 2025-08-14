<template>
	<div class="container px-4 safe-area-inset">
		<TopHeader :title="$t('monitoring.title')" />
		<div class="row">
			<main class="col-12">
				<div class="header-outer sticky-top">
					<div class="container px-4">
						<div class="row py-3 py-sm-3 d-flex flex-column flex-sm-row gap-3 gap-lg-0 mb-lg-2">
							<div class="col-lg-5 d-flex mb-lg-0">
								<SelectGroup
									id="monitoringType"
									class="w-100 d-flex"
									:options="typeOptions"
									large
									:modelValue="activeTab"
									@update:model-value="setActiveTab"
								/>
							</div>
							<div class="col-lg-6 offset-lg-1">
								<div class="d-flex justify-content-lg-end">
									<DateSelector
										:selectedDate="selectedDate"
										@update-date="updateSelectedDate"
									/>
								</div>
							</div>
						</div>
					</div>
				</div>

				<h3 class="fw-normal my-0 d-flex gap-3 flex-wrap d-flex align-items-baseline overflow-hidden">
					<span class="d-block no-wrap text-truncate">
						{{ historyTitle }}
					</span>
					<small class="d-block no-wrap text-truncate">{{ historySubTitle }}</small>
				</h3>

				<!-- Site Power Tab -->
				<div v-if="activeTab === 'sitepower'">
					<div v-if="loading.sitepower" class="text-center py-5">
						<div class="spinner-border text-primary mb-3" role="status">
							<span class="visually-hidden">Loading...</span>
						</div>
						<h5 class="text-muted">Loading Power Data...</h5>
						<p class="text-muted small">Fetching data for {{ selectedDate }}</p>
					</div>
					<div v-else-if="error.sitepower" class="alert alert-danger">
						{{ error.sitepower }}
					</div>
					<div v-else>
						<div v-if="sitePowerData.length === 0" class="text-center py-5 text-muted">
							<i class="fas fa-chart-line fa-3x mb-3 text-muted"></i>
							<h5>No Data Available</h5>
							<p>No power data found for {{ selectedDate }}.</p>
							<p class="small">Try selecting a different date or check if the system is collecting data.</p>
						</div>
						<PowerChart
							v-else
							:data="chartData"
							:dataType="'sitepower'"
							:selectedDate="selectedDate"
							class="mb-5"
						/>

						<!-- Statistics Section -->
						<div class="row align-items-start mb-5">
							<div class="col-12">
								<h3 class="fw-normal my-4">{{ $t('monitoring.statistics') }}</h3>
								<div class="row text-center">
									<div class="col-md-3 mb-3">
										<div class="stat-item">
											<div class="stat-label">{{ $t('monitoring.stats.max').toUpperCase() }}</div>
											<div class="stat-value">{{ formatPower(maxPower) }}</div>
										</div>
									</div>
									<div class="col-md-3 mb-3">
										<div class="stat-item">
											<div class="stat-label">{{ $t('monitoring.stats.min').toUpperCase() }}</div>
											<div class="stat-value">{{ formatPower(minPower) }}</div>
										</div>
									</div>
									<div class="col-md-3 mb-3">
										<div class="stat-item">
											<div class="stat-label">{{ $t('monitoring.stats.avg').toUpperCase() }}</div>
											<div class="stat-value">{{ formatPower(averagePower) }}</div>
										</div>
									</div>
									<div class="col-md-3 mb-3">
										<div class="stat-item">
											<div class="stat-label">{{ $t('monitoring.stats.total').toUpperCase() }}</div>
											<div class="stat-value">{{ sitePowerData.length }}</div>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>

				<!-- Battery Charging Tab -->
				<div v-if="activeTab === 'battery'">
					<div class="text-center py-5 text-muted mb-5">
						<p>Battery charging data will be available soon.</p>
					</div>
				</div>

				<!-- Export Button -->
				<div class="d-flex gap-2 mt-1 mb-5">
					<button class="btn btn-outline-secondary" @click="exportData" :disabled="!sitePowerData.length">
						Export CSV
					</button>
				</div>
			</main>
		</div>
	</div>
</template>

<script lang="ts">
import Header from "../components/Top/Header.vue";
import DateSelector from "../components/Monitoring/DateSelector.vue";
import PowerChart from "../components/Monitoring/PowerChart.vue";
import SelectGroup from "../components/Helper/SelectGroup.vue";
import api from "../api";
import { defineComponent } from 'vue';

export default defineComponent({
	name: 'Monitoring',
	components: {
		TopHeader: Header,
		DateSelector,
		PowerChart,
		SelectGroup,
	},
	data() {
		return {
			activeTab: 'sitepower',
			selectedDate: new Date().toISOString().split('T')[0], // 移到data中作为内部状态
			sitePowerData: [],
			lastUpdate: new Date(),
			loading: {
				sitepower: false,
			},
			error: {
				sitepower: null,
			},
		};
	},
	computed: {
		typeOptions() {
			return [
				{ name: this.$t('monitoring.dataType.sitepower'), value: 'sitepower' },
				{ name: this.$t('monitoring.dataType.battery'), value: 'battery' }
			];
		},
		historyTitle() {
			return this.activeTab === 'sitepower'
				? this.$t('monitoring.chartTitle.sitepower')
				: this.$t('monitoring.chartTitle.battery');
		},
		historySubTitle() {
			const date = new Date(this.selectedDate);
			const formattedDate = date.toLocaleDateString('en-US', {
				year: 'numeric',
				month: '2-digit',
				day: '2-digit'
			});
			if (this.activeTab === 'sitepower' && this.sitePowerData.length > 0) {
				return `${formattedDate} • ${this.sitePowerData.length} data points • ⌀ ${this.formatPower(this.averagePower)}`;
			}
			return formattedDate;
		},
		formattedSelectedDate() {
			const date = new Date(this.selectedDate);
			return date.toLocaleDateString('en-US', {
				year: 'numeric',
				month: '2-digit',
				day: '2-digit'
			});
		},
		chartData() {
			if (!this.sitePowerData || this.sitePowerData.length === 0) {
				return [];
			}
			return this.sitePowerData.map(record => ({
				time: record.createdAt,
				power: record.powerKW || 0
			}));
		},
		maxPower() {
			if (!this.sitePowerData || this.sitePowerData.length === 0) return null;
			return Math.max(...this.sitePowerData.map(record => record.powerKW));
		},
		minPower() {
			if (!this.sitePowerData || this.sitePowerData.length === 0) return null;
			return Math.min(...this.sitePowerData.map(record => record.powerKW));
		},
		averagePower() {
			if (!this.sitePowerData || this.sitePowerData.length === 0) return null;
			const total = this.sitePowerData.reduce((sum, record) => sum + (record.powerKW || 0), 0);
			return total / this.sitePowerData.length;
		},
		isRefreshing() {
			return this.loading.sitepower;
		},
	},
	methods: {
		formatPower(power) {
			if (power === null || power === undefined) return 'N/A';
			return `${power.toFixed(2)} kW`;
		},
		setActiveTab(tab) {
			this.activeTab = tab;
		},
		updateSelectedDate(date) {
			this.selectedDate = date;
			// watch会自动触发loadSitePowerData，所以这里不需要手动调用
		},
		async loadSitePowerData() {
			this.loading.sitepower = true;
			this.error.sitepower = null;

			try {
				const params = {
					site: 'Zuhause' // Use the site name from the API response
				};

				// Set time range for the selected date
				if (this.selectedDate) {
					const date = new Date(this.selectedDate);
					const from = new Date(date);
					from.setHours(0, 0, 0, 0);
					const to = new Date(date);
					to.setHours(23, 59, 59, 999);

					// Convert to ISO string format as expected by the API
					params['from'] = from.toISOString();
					params['to'] = to.toISOString();
				}

				// 添加超时和缓存控制
				const response = await api.get('/sitepower/records', {
					params,
					timeout: 10000, // 10秒超时
					headers: {
						'Cache-Control': 'max-age=60' // 缓存1分钟
					}
				});
				this.sitePowerData = response.data.records || [];
				this.lastUpdate = new Date();
			} catch (err) {
				if (err.code === 'ECONNABORTED') {
					this.error.sitepower = 'Request timeout. Please check your connection and try again.';
				} else {
					this.error.sitepower = err.message || 'Failed to load site power data';
				}
				console.error('Error loading site power data:', err);
			} finally {
				this.loading.sitepower = false;
			}
		},
		async refreshData() {
			await this.loadSitePowerData();
		},
		exportData() {
			if (!this.sitePowerData.length) return;

			const csvContent = [
				['Timestamp', 'Site Title', 'Power (kW)'],
				...this.sitePowerData.map(record => [
					record.createdAt,
					record.siteTitle,
					record.powerKW
				])
			].map(row => row.join(',')).join('\n');

			const blob = new Blob([csvContent], { type: 'text/csv' });
			const url = window.URL.createObjectURL(blob);
			const link = document.createElement('a');
			link.href = url;
			link.download = `site-power-${this.selectedDate}.csv`;
			link.click();
			window.URL.revokeObjectURL(url);
		},
	},
	watch: {
		selectedDate: {
			handler(newDate, oldDate) {
				if (newDate !== oldDate) {
					this.loadSitePowerData();
				}
			},
			immediate: false // 不立即执行，避免与mounted重复
		}
	},
	head() {
		return {
			title: this.$t('monitoring.title'),
			titleTemplate: '%s'
		};
	},
	mounted() {
		this.loadSitePowerData();
	},
});
</script>

<style scoped>
.header-outer {
	--vertical-shift: 0rem;
	left: 0;
	right: 0;
	top: max(0rem, env(safe-area-inset-top)) !important;
	margin: 0 calc(calc(1.5rem + var(--vertical-shift)) * -1);
	-webkit-backdrop-filter: blur(35px);
	backdrop-filter: blur(35px);
	background-color: #0000;
	box-shadow: 0 1px 8px 0px var(--evcc-background);
}

@media (min-width: 576px) {
	.header-outer {
		--vertical-shift: calc((100vw - 540px) / 2);
	}
}

@media (min-width: 768px) {
	.header-outer {
		--vertical-shift: calc((100vw - 720px) / 2);
	}
}

@media (min-width: 992px) {
	.header-outer {
		--vertical-shift: calc((100vw - 960px) / 2);
	}
}

@media (min-width: 1200px) {
	.header-outer {
		--vertical-shift: calc((100vw - 1140px) / 2);
	}
}

@media (min-width: 1400px) {
	.header-outer {
		--vertical-shift: calc((100vw - 1320px) / 2);
	}
}

.stat-item {
	padding: 1.5rem;
	border: 1px solid var(--evcc-box-border);
	border-radius: 12px;
	background-color: var(--evcc-background);
	transition: all 0.2s ease;
}

.stat-item:hover {
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	transform: translateY(-1px);
}

.stat-label {
	font-size: 0.75rem;
	font-weight: 600;
	color: var(--evcc-gray);
	text-transform: uppercase;
	letter-spacing: 0.5px;
	margin-bottom: 0.75rem;
}

.stat-value {
	font-size: 1.75rem;
	font-weight: 700;
	color: var(--evcc-default-text);
	line-height: 1.2;
}

.no-wrap {
	white-space: nowrap;
}

@media (max-width: 768px) {
	.stat-item {
		padding: 1rem;
		margin-bottom: 0.75rem;
	}

	.stat-value {
		font-size: 1.5rem;
	}
}
</style>