<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<XColumn :column="column" :isStacked="isStacked" :refresher="() => reloadTimeline()">
	<template #header><i class="ti ti-mail" style="margin-right: 8px;"></i>{{ column.name || i18n.ts._deck._columns.direct }}</template>

	<MkNotesTimeline ref="tlComponent" :pagination="pagination"/>
</XColumn>
</template>

<script lang="ts" setup>
import { ref, useTemplateRef } from 'vue';
import XColumn from './column.vue';
import type { Column } from '@/deck.js';
import MkNotesTimeline from '@/components/MkNotesTimeline.vue';
import { i18n } from '@/i18n.js';

defineProps<{
	column: Column;
	isStacked: boolean;
}>();

const pagination = {
	endpoint: 'notes/mentions' as const,
	limit: 10,
	params: {
		visibility: 'specified',
	},
};

const tlComponent = useTemplateRef('tlComponent');

function reloadTimeline() {
	return new Promise<void>((res) => {
		tlComponent.value?.reload().then(() => {
			res();
		});
	});
}
</script>
