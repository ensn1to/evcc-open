<template>
	<div class="battery-card d-flex flex-column pt-4 pb-2 px-3 px-sm-4 mx-2 mx-sm-0 h-100" data-testid="battery-card">
		<div class="d-block d-sm-flex justify-content-between align-items-center mb-3">
			<div class="d-flex justify-content-between align-items-center mb-3 text-truncate">
				<h3 class="me-2 mb-0 text-truncate d-flex">
					<div class="text-truncate">Home Battery</div>
				</h3>
			</div>
		</div>

		<div class="details d-flex align-items-start mb-2">
			<div>
				<div class="d-flex align-items-center">
					<div class="root mb-2 text-nowrap text-truncate-xs-only">
						<div class="mb-2 label text-truncate-xs-only text-start">电池功率</div>
						<h3 class="value m-0 justify-content-start">
							<span>{{ formattedPower }}</span>
						</h3>
					</div>
					<shopicon-regular-battery
						class="text-evcc opacity-transiton"
						:class="batteryIconClass"
						size="m"
						data-shopicon="true"
					>
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48">
							<path d="M0 0h48v48H0z" fill="none"></path>
							<path d="M32 6H16c-2.2 0-4 1.8-4 4v28c0 2.2 1.8 4 4 4h16c2.2 0 4-1.8 4-4V10c0-2.2-1.8-4-4-4zm0 32H16V10h16v28zM20 2h8v4h-8z"></path>
							<rect v-if="batterySoc > 0" :x="18" :y="38 - (batterySoc * 0.24)" :width="12" :height="batterySoc * 0.24" fill="currentColor"></rect>
						</svg>
					</shopicon-regular-battery>
				</div>
			</div>
		</div>

		<hr class="divider">

		<div class="battery-info pt-4 flex-grow-1 d-flex flex-column justify-content-end">
			<div class="d-flex justify-content-between mb-3 align-items-center" data-testid="battery-status">
				<h4 class="d-flex align-items-center m-0 flex-grow-1 overflow-hidden">
				</h4>
			</div>

			<div class="battery-soc mt-1 mb-4">
				<div class="d-flex align-items-center gap-2 mb-2">
					<div class="battery-status evcc-gray" data-testid="battery-status-text">SOC</div>
					<div class="progress flex-grow-1">
						<div 
							class="progress-bar bg-success"
							:class="{ 'progress-bar-striped progress-bar-animated': isCharging }"
							role="progressbar" 
							:style="`width: ${batterySoc}%; transition: width var(--evcc-transition-fast) linear;`"
						>
							{{ formattedSoc }}
						</div>
					</div>
				</div>
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
					<div class="value m-0 d-block align-items-baseline justify-content-center text-decoration-underline">
						{{ batteryStatusText }}
					</div>
				</div>

				<div class="root flex-grow-1 text-end" data-testid="battery-soc">
					<div class="mb-2 label text-truncate-xs-only text-end">SOC</div>
					<h3 class="value m-0 justify-content-end">
						<span class="text-decoration-underline text-gray fw-normal" data-testid="battery-soc-value">
							<span>{{ formattedSoc }}</span>
						</span>
					</h3>
				</div>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import formatter from "@/mixins/formatter";

export default defineComponent({
	name: 'BatteryCard',
	mixins: [formatter],
	props: {
		batteryPower: { type: Number, default: 0 },
		batterySoc: { type: Number, default: 0 },
		batteryMode: { type: String, default: '' },
		batteryConfigured: { type: Boolean, default: false }
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
				return 'Battery not configured';
			}
			if ((this as any).batteryPower > 0) {
				return 'Battery discharging';
			} else if ((this as any).batteryPower < 0) {
				return 'Battery charging';
			} else {
				return 'Battery idle';
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
	margin: 1rem 0;
}

.battery-soc .progress {
	height: 1rem;
	border-radius: 0.25rem;
	background-color: var(--evcc-box);
}

.battery-soc .progress-bar {
	border-radius: 0.25rem;
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