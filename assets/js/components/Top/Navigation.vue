<template>
	<div>
		<button
			id="topNavigatonDropdown"
			type="button"
			data-bs-toggle="dropdown"
			aria-expanded="false"
			class="btn btn-sm btn-outline-secondary position-relative border-0 menu-button"
			data-testid="topnavigation-button"
		>
			<span
				v-if="showRootBadge"
				class="position-absolute top-0 start-100 translate-middle p-2 rounded-circle"
				:class="badgeClass"
			>
				<span class="visually-hidden">action required</span>
			</span>
			<shopicon-regular-menu></shopicon-regular-menu>
		</button>
		<ul
			class="dropdown-menu dropdown-menu-end"
			aria-labelledby="topNavigatonDropdown"
			data-testid="topnavigation-dropdown"
		>
			<li>
				<router-link class="dropdown-item" to="/sessions" active-class="active">
					{{ this.t("header.sessions") }}
				</router-link>
			</li>
			<li>
				<router-link class="dropdown-item" to="/monitoring" active-class="active">
					{{ this.t("header.monitoring") }}
				</router-link>
			</li>
			<li><hr class="dropdown-divider" /></li>
			<li>
				<button
					type="button"
					class="dropdown-item"
					data-testid="topnavigation-settings"
					@click="openSettingsModal"
				>
					{{ this.t("settings.title") }}
				</button>
			</li>
			<li v-if="batteryModalAvailable">
				<button
					type="button"
					class="dropdown-item"
					data-testid="topnavigation-battery"
					@click="openBatterySettingsModal"
				>
					{{ this.t("batterySettings.modalTitle") }}
				</button>
			</li>
			<li v-if="forecastAvailable">
				<button
					type="button"
					class="dropdown-item"
					data-testid="topnavigation-forecast"
					@click="openForecastModal"
				>
					{{ this.t("forecast.modalTitle") }}
				</button>
			</li>
			<li>
				<router-link class="dropdown-item" to="/config" active-class="active">
					<span
						v-if="showConfigBadge"
						class="d-inline-block p-1 rounded-circle bg-warning rounded-circle"
						:class="badgeClass"
					></span>
					{{ this.t("config.main.title") }}
				</router-link>
			</li>
			<li>
				<router-link class="dropdown-item" to="/log" active-class="active">
					{{ this.t("log.title") }}
				</router-link>
			</li>
			<li>
						<router-link class="dropdown-item" to="/optimize" active-class="active">
							Optimize 🧪
						</router-link>
					</li>
			<li><hr class="dropdown-divider" /></li>
			<template v-if="providerLogins.length > 0">
				<li>
					<h6 class="dropdown-header">{{ this.t("header.authProviders.title") }}</h6>
				</li>
				<li v-for="l in providerLogins" :key="l.title">
					<button
						type="button"
						class="dropdown-item"
						@click="handleProviderAuthorization(l)"
					>
						<span
							class="d-inline-block p-1 rounded-circle border border-light rounded-circle"
							:class="l.authenticated ? 'bg-success' : 'bg-warning'"
						></span>
						{{ l.title }}
					</button>
				</li>
				<li><hr class="dropdown-divider" /></li>
			</template>
			<li>
				<button type="button" class="dropdown-item" @click="openHelpModal">
					<span>{{ this.t("header.needHelp") }}</span>
				</button>
			</li>
			<!-- <li>
				<a class="dropdown-item d-flex" href="https://evcc.io/" target="_blank">
					<span>evcc.io</span>
					<shopicon-regular-newtab
						size="s"
						class="ms-2 external"
					></shopicon-regular-newtab>
				</a>
			</li> -->
			<li v-if="isApp">
				<button type="button" class="dropdown-item" @click="openNativeSettings">
					{{ this.t("header.nativeSettings") }}
				</button>
			</li>
			<li v-if="showLogout">
				<button type="button" class="dropdown-item" @click="logout">
					{{ this.t("header.logout") }}
				</button>
			</li>
		</ul>
	</div>
</template>

<script lang="ts">
import Modal from "bootstrap/js/dist/modal";
import Dropdown from "bootstrap/js/dist/dropdown";
import "@h2d2/shopicons/es/regular/gift";
import "@h2d2/shopicons/es/regular/moonstars";
import "@h2d2/shopicons/es/regular/menu";
import "@h2d2/shopicons/es/regular/newtab";
import { logout, isLoggedIn, openLoginModal } from "../Auth/auth";
import baseAPI from "./baseapi";
import { isApp, sendToApp } from "@/utils/native";
import { isUserConfigError } from "@/utils/fatal";
import { defineComponent, type PropType, getCurrentInstance } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
import store from "@/store";
import type { FatalError, Sponsor, AuthProviders, Battery, Forecast, EvOpt } from "@/types/evcc";

interface Provider {
	title: string;
	authenticated: boolean;
	loginPath: string;
	logoutPath: string;
}

export default defineComponent({
	name: "TopNavigation",
	data() {
		const { t } = useI18n();
		const router = useRouter();
		const instance = getCurrentInstance();
		return {
			isApp: isApp(),
			dropdown: null as Dropdown | null,
			t,
			router,
			instance
		};
	},
	computed: {
		batteryConfigured(): boolean {
			return store.state.battery?.length > 0 || false;
		},
		providerLogins(): Provider[] {
			return Object.entries(store.state.authProviders || {}).map(
				([title, { authenticated, id }]: [string, any]) => ({
					title,
					authenticated,
					loginPath: "providerauth/login?id=" + id,
					logoutPath: "providerauth/logout?id=" + id,
				})
			);
		},
		loginRequired(): boolean {
			const authProviders = store.state.authProviders || {};
			return Object.values(authProviders).some((p: any) => !p.authenticated);
		},
		showConfigBadge(): boolean {
			const userConfigError = isUserConfigError(store.state.fatal || []);
			return store.state.sponsor?.expiresSoon || userConfigError;
		},
		showRootBadge(): boolean {
			return this.loginRequired || this.showConfigBadge;
		},
		badgeClass(): string {
			if ((store.state.fatal || []).length > 0) {
				return "bg-danger";
			}
			return "bg-warning";
		},
		batteryModalAvailable(): boolean {
			return this.batteryConfigured;
		},
		forecastAvailable(): boolean {
			const { grid, solar, co2 } = store.state.forecast || {};
			return !!(grid || solar || co2);
		},
		optimizeAvailable(): boolean {
			// 简化实现，始终返回true确保显示
			return true;
		},
		showLogout(): boolean {
			return isLoggedIn();
		},
	},
	mounted() {
		this.$nextTick(() => {
			// 尝试多次初始化Dropdown，确保元素已加载
			const initDropdown = () => {
				const element = document.getElementById("topNavigatonDropdown");
				if (element) {
					// 先移除可能存在的旧实例
					if (this.dropdown) {
						this.dropdown.dispose();
						this.dropdown = null;
					}
					
					// 创建新的Dropdown实例
					this.dropdown = new Dropdown(element);
					
					// 添加可靠的点击事件监听器
					element.removeEventListener("click", this.handleDropdownClick);
					element.addEventListener("click", this.handleDropdownClick);
					
					console.log("Dropdown initialized successfully");
				} else {
					// 如果第一次没找到元素，稍后再试
					setTimeout(initDropdown, 100);
				}
			};
			
			initDropdown();
		});
	},
	unmounted() {
		if (this.dropdown) {
			this.dropdown.dispose();
		}
	},
	methods: {
		async handleProviderAuthorization(provider: Provider): Promise<void> {
			const { title, authenticated, loginPath, logoutPath } = provider;
			if (!authenticated) {
				try {
					const response = await baseAPI.get(loginPath);
					window.location.href = response.data.loginUri;
				} catch (error: any) {
					console.error(error);
					alert(`Failed to login: ${error.response?.data}`);
				}
			} else {
				if (
					window.confirm(
						this.t("header.authProviders.confirmLogout", { title })
					)
				) {
					try {
						await baseAPI.get(logoutPath);
					} catch (error: any) {
						console.error(error);
						alert(`Failed to logout: ${error.response?.data}`);
					}
				}
			}
		},
		openSettingsModal(): void {
			const modal = Modal.getOrCreateInstance(
				document.getElementById("globalSettingsModal") as HTMLElement
			);
			modal.show();
		},
		openHelpModal(): void {
			const modal = Modal.getOrCreateInstance(
				document.getElementById("helpModal") as HTMLElement
			);
			modal.show();
		},
		openBatterySettingsModal(): void {
			const modal = Modal.getOrCreateInstance(
				document.getElementById("batterySettingsModal") as HTMLElement
			);
			modal.show();
		},
		openForecastModal(): void {
			const modal = Modal.getOrCreateInstance(
				document.getElementById("forecastModal") as HTMLElement
			);
			modal.show();
		},
		openNativeSettings(): void {
			sendToApp({ type: "settings" });
		},
		async login(): Promise<void> {
			openLoginModal();
		},
		async logout(): Promise<void> {
			await logout();
			this.router.push({ path: "/" });
		},
		
		// 下拉菜单点击处理函数
		handleDropdownClick(e: MouseEvent): void {
			e.preventDefault();
			e.stopPropagation();
			if (this.dropdown) {
				this.dropdown.toggle();
			}
		},
	},
});
</script>
<style scoped>
.menu-button {
	margin-right: -0.7rem;
	cursor: pointer;
}
.external {
	width: 18px;
	height: 20px;
}
.dropdown-menu {
	/* above sticky, below modal https://getbootstrap.com/docs/5.3/layout/z-index/ */
	z-index: 1045 !important;
	display: none; /* 确保初始状态为隐藏 */
}
.dropdown-menu.show {
	display: block; /* 确保激活状态下显示 */
}
/* 确保菜单项在各种状态下都能正常显示 */
.dropdown-item {
	visibility: visible !important;
	opacity: 1 !important;
}
</style>
