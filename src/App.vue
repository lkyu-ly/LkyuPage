<script setup>
import { SpeedInsights } from '@vercel/speed-insights/vue';
import { RouterView } from 'vue-router';
</script>

<template>
	<AppProvider>
		<AppContainer>
			<RouterView />
		</AppContainer>
	</AppProvider>
	<SpeedInsights />
</template>
