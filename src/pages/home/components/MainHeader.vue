<script setup lang="ts">
import { RouterLink } from 'vue-router';

const route = useRoute();
const settingsActived = ref(false);

function toggleSetting() {
	return (settingsActived.value = route.path === '/setting');
}

function getIconClass(routeName: string) {
	return {
		'text-$primary-c opacity-100': routeName === route.name,
	};
}
</script>

<template>
	<div flex justify-between py-0 px="12 md:24 lg:48" :class="{ 'logo-head': 'logo_head' }">
		<RouterLink to="/">
			<div text="$primary-c" flex-center text-16 style="font-weight: bold">
				<img
					decoding="async"
					loading="lazy"
					w-24
					src="/favicon.ico"
					inline-block
					text-32
					transition
					duration-300
					hover="opacity-70"
				/>
			</div>
		</RouterLink>
		<div flex gap-x-8>
			<RouterLink
				:class="getIconClass('setting')"
				:to="toggleSetting() ? '/' : '/setting'"
				i-carbon:settings
				icon-btn
				w-24
				h-24
			/>
		</div>
	</div>
</template>
