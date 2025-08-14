<template>
	<div class="battery-card d-flex flex-column pt-4 pb-2 px-3 px-sm-4 mx-2 mx-sm-0 h-100" data-testid="battery-card">
		<div class="d-block d-sm-flex justify-content-between align-items-center mb-3">
			<div class="d-flex justify-content-between align-items-center mb-3 text-truncate">
				<h3 class="me-2 mb-0 text-truncate d-flex">
					<div class="text-truncate">Battery</div>
				</h3>
			</div>
			<button 
				v-if="batteryConfigured"
				type="button" 
				class="btn btn-sm btn-outline-secondary position-relative border-0 p-2 evcc-gray d-none d-sm-block ms-2" 
				@click="openBatterySettingsModal"
				title="Battery Settings"
			>
				<shopicon-regular-adjust size="s"></shopicon-regular-adjust>
			</button>
		</div>

		<div class="details d-flex flex-wrap justify-content-between">
			<div class="root flex-grow-1" data-testid="battery-power">
				<div class="mb-2 label text-truncate-xs-only text-start">功率</div>
				<h3 class="value m-0 justify-content-start">
					<span>{{ formattedPower }}</span>
					<div class="extraValue text-nowrap">&nbsp;</div>
				</h3>
			</div>

			<div class="text-center flex-grow-1">
				<div class="mb-2 label text-truncate-xs-only text-center">状态</div>
				<div class="value m-0 d-block align-items-baseline justify-content-center" style="font-size: 0.875rem;">
				{{ batteryStatusText }}
			</div>
			</div>

			<div class="root flex-grow-1 text-end" data-testid="battery-soc">
				<div class="mb-2 label text-truncate-xs-only text-end">Capacity</div>
				<h3 class="value m-0 justify-content-end">
					<span class="text-gray fw-normal" data-testid="battery-soc-value">
						<span style="font-size: 0.875rem;">{{ batteryCapacityText }}</span>
					</span>
				</h3>
			</div>
		</div>

		<hr class="divider">

		<div class="battery-info pt-2 flex-grow-1 d-flex flex-column justify-content-end">
			<div class="d-flex justify-content-between mb-3 align-items-center" data-testid="battery-status">
				<h4 class="d-flex align-items-center m-0 flex-grow-1 overflow-hidden">
					<div class="battery-status evcc-gray" data-testid="battery-status-text">SOC</div>
				</h4>
			</div>

			<div class="battery-soc mt-1 mb-4">
				<div class="d-flex align-items-center gap-2 mb-2">
					<div class="progress flex-grow-1">
						<div 
							class="progress-bar bg-success"
							role="progressbar" 
							:style="`width: ${batterySoc}%; transition: width var(--evcc-transition-fast) linear;`"
						>
							{{ formattedSoc }}
						</div>
					</div>
				</div>
			</div>


		</div>
	</div>
</template>

<script lang="ts">
import '@h2d2/shopicons/es/regular/adjust';
import { defineComponent } from 'vue';
import formatter from '@/mixins/formatter';
import { Modal } from 'bootstrap';

export default defineComponent({
	name: 'BatteryCard',
	mixins: [formatter],
	props: {
		batteryPower: { type: Number, default: 0 },
		batterySoc: { type: Number, default: 0 },
		batteryMode: { type: String, default: '' },
		batteryConfigured: { type: Boolean, default: false },
		batteryCapacity: { type: Number, default: 13.4 }
	},
	computed: {
		formattedPower() {
			return (this as any).fmtW(Math.abs((this as any).batteryPower));
		},
		formattedSoc() {
			return (this as any).fmtPercentage((this as any).batterySoc);
		},
		batteryStatusText() {
			if (!(this as any).batteryConfigured) {
				return 'not configured';
			}
			if ((this as any).batteryPower > 0) {
				return 'discharging';
			} else if ((this as any).batteryPower < 0) {
				return 'charging';
			} else {
				return 'idle';
			}
		},
		isCharging() {
			return (this as any).batteryPower < 0;
		},
		batteryIconClass() {
			if ((this as any).isCharging) {
				return 'opacity-100 text-success';
			} else if ((this as any).batteryPower > 0) {
				return 'opacity-100 text-warning';
			} else {
				return 'opacity-100';
			}
		},
		batteryCapacityText() {
			const currentEnergy = ((this as any).batteryCapacity / 100) * (this as any).batterySoc;
			const totalEnergy = (this as any).batteryCapacity;
			return `${currentEnergy.toFixed(1)} kWh of ${totalEnergy.toFixed(1)} kWh`;
		}
	},
	methods: {
		openBatterySettingsModal() {
			const modal = Modal.getOrCreateInstance(
				document.getElementById('batterySettingsModal') as HTMLElement
			);
			modal.show();
		}
	}
});
</script>

<style scoped>
.battery-card {
	border-radius: 2rem;
	color: var(--evcc-default-text);
	background: var(--evcc-box);
}

.root {
	display: flex;
	flex-direction: column;
}

.label {
	font-size: 0.875rem;
	color: var(--evcc-gray);
	font-weight: 500;
}

.value {
	font-size: 1.25rem;
	font-weight: 600;
	color: var(--evcc-default-text);
}

.divider {
	border: none;
	border-top: 1px solid var(--evcc-box);
	margin: 0.5rem 0;
}

.battery-soc .progress {
	height: 1.5rem;
	border-radius: 0.25rem;
	background-color: var(--evcc-box);
}

.battery-soc .progress-bar {
	border-radius: 0.25rem;
}

.battery-status {
	font-weight: normal;
	font-size: 0.875rem;
}

.icon {
	width: 1.5rem;
	height: 1.5rem;
}

.icon--s {
	width: 1rem;
	height: 1rem;
}

.opacity-transiton {
	transition: opacity 0.2s ease;
}

.text-truncate-xs-only {
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
}

@media (max-width: 575.98px) {
	.text-truncate-xs-only {
		max-width: 150px;
	}
}
</style>