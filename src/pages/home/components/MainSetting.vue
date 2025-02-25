<script setup lang="ts">
import { toggleSiteSytle } from '@/composables/dark';
import preset from '@/preset.json';
import router from '@/router';
import type { Category, Settings } from '@/types';
import type { ThemeSetting } from '@/utils';
import { iconStyleList, searchList, siteStyleList, themeList } from '@/utils';
import ResetModal from './ResetModal.vue';
import SettingSelection from './SettingSelection.vue';

const resetStore = useResetModalStore();
const settingStore = useSettingStore();
const renderStore = useRenderStore();

/* ThemeSetting */
function renderThemeLabel(option: ThemeSetting): VNode {
	const currentTheme = themeList.find(item => item.enName === option.enName)!;
	const buttonColor = currentTheme!.value.buttonC;
	const darkConfig = isDark.value ? { style: { color: '#ffffff' } } : {};
	return h('div', { class: 'flex items-center gap-x-8' }, [
		h('div', {
			class: 'w-16 h-16 circle border-1 border-fff',
			style: { backgroundColor: buttonColor },
		}),
		h('div', darkConfig, option.name),
	]);
}

/* render color */
function renderColor(option: any): VNode {
	const darkConfig = isDark.value ? { style: { color: '#ffffff' } } : {};
	return h('div', { class: 'flex items-center gap-x-8' }, [h('div', darkConfig, option.name)]);
}

/* Icon Style */

/* import and export */
interface CacheData {
	data: Category[];
	settings: Settings;
}

const siteStore = useSiteStore();

function exportData() {
	const data = {
		data: siteStore.data,
		settings: settingStore.settings,
	};

	// 使用 JSON.stringify 并添加缩进参数
	const jsonStr = JSON.stringify(data, null, 4); // 使用 4 个空格进行缩进

	const blob = new Blob([jsonStr], { type: 'application/json' });

	const url = URL.createObjectURL(blob);
	const a = document.createElement('a');
	a.download = `LKYUPAGE_Data_${new Date().toLocaleString()}.json`;
	a.href = url;
	document.body.appendChild(a);
	a.click();
	URL.revokeObjectURL(url);
}

function importData() {
	const inputElement = document.createElement('input');
	inputElement.type = 'file';
	inputElement.accept = '.json';
	inputElement.addEventListener('change', async () => {
		const file = inputElement.files?.[0];
		if (file) {
			try {
				const jsonStr = await file.text();
				const data = JSON.parse(jsonStr) as CacheData;
				if (!data.data || !data.settings) {
					throw new Error('非法的数据文件');
				}
				loadData(data);
			} catch (error) {
				window.$message.error('请导入合法的数据文件', { duration: 2000 });
			}
		}
	});
	inputElement.click();
}

function resetData() {
	resetStore.title = '重置确认';
	resetStore.content = '是否确认要重置所有设置?';
	resetStore.resetVisible = true;
	resetStore.afterCommit = () => {
		router.back();
	};
	resetStore.finishCommit = () => {
		const clonedPreset = JSON.parse(JSON.stringify(preset));
		const data = clonedPreset as CacheData;
		loadData(data);
		window.$message.success('重置成功', { duration: 2000 });
	};
}

function loadData(data: any) {
	siteStore.setData(data.data);
	settingStore.setSettings(data.settings);
	toggleTheme(data.settings.theme);
	toggleSiteSytle();
	siteStore.cateIndex = 0;
	renderStore.refreshSiteGroupList();
}
</script>

<template>
	<section v-if="settingStore.isSetting" px="md:60 lg:120">
		<div grid grid-cols-2 gap-24 lg:grid-cols-2 md:grid-cols-2>
			<SettingSelection
				v-model="settingStore.settings.theme"
				title="主题风格"
				:options="themeList"
				:render-label="renderThemeLabel"
				label-field="name"
				value-field="enName"
				:on-update-value="(theme: string) => toggleTheme(theme)"
			/>
			<SettingSelection
				v-model="settingStore.settings.search"
				title="搜索引擎"
				:options="searchList"
				:render-label="renderColor"
				label-field="name"
				value-field="enName"
				:on-update-value="(enName: string) => settingStore.setSettings({ search: enName })"
			/>
			<SettingSelection
				v-model="settingStore.settings.iconStyle"
				title="图标风格"
				:options="iconStyleList"
				:render-label="renderColor"
				label-field="name"
				value-field="enName"
				:on-update-value="(enName: string) => settingStore.setSettings({ iconStyle: enName })"
			/>
			<SettingSelection
				v-model="settingStore.settings.siteStyle"
				title="色彩模式"
				:options="siteStyleList"
				:render-label="renderColor"
				label-field="name"
				value-field="enName"
				:on-update-value="(enName: string) => {
          settingStore.setSettings({ siteStyle: enName })
          toggleSiteSytle()
        }"
			/>
		</div>
		<div mt-24 flex justify-center gap-x-24>
			<n-button @click="resetData">重置数据</n-button>
			<n-button @click="importData">导入数据</n-button>
			<n-button @click="exportData">导出数据</n-button>
		</div>
		<div my-24 flex-center gap-x-24>
			<n-button type="primary" text-color="#ffffff" size="large" @click="$router.back()">
				返回
			</n-button>
		</div>
	</section>
	<ResetModal />
</template>
