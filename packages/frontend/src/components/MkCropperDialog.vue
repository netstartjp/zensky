<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkModalWindow
	ref="dialogEl"
	:width="800"
	:height="500"
	:scroll="false"
	:withOkButton="true"
	@close="cancel()"
	@ok="ok()"
	@closed="emit('closed')"
>
	<template #header>{{ i18n.ts.cropImage }}</template>
	<div class="mk-cropper-dialog" :style="`--vw: 100%; --vh: 100%;`">
		<Transition name="fade">
			<div v-if="loading" class="loading">
				<MkLoading/>
			</div>
		</Transition>
		<div class="container">
			<img ref="imgEl" :src="imgUrl" style="display: none;" @load="onImageLoad">
		</div>
	</div>
</MkModalWindow>
</template>

<script lang="ts" setup>
import { onMounted, useTemplateRef, ref } from 'vue';
import * as Misskey from 'misskey-js';
import Cropper from 'cropperjs';
import tinycolor from 'tinycolor2';
import MkModalWindow from '@/components/MkModalWindow.vue';
import * as os from '@/os.js';
import { i18n } from '@/i18n.js';

const props = defineProps<{
	imageFile: File | Blob;
	aspectRatio: number | null;
	uploadFolder?: string | null;
}>();

const emit = defineEmits<{
	(ev: 'ok', cropped: File | Blob): void;
	(ev: 'cancel'): void;
	(ev: 'closed'): void;
}>();

const imgUrl = URL.createObjectURL(props.imageFile);
const dialogEl = useTemplateRef('dialogEl');
const imgEl = useTemplateRef('imgEl');
let cropper: Cropper | null = null;
const loading = ref(true);

const ok = async () => {
	const promise = new Promise<Misskey.entities.DriveFile>(async (res) => {
		const croppedImage = await cropper?.getCropperImage();
		const croppedSection = await cropper?.getCropperSelection();

		// 拡大率を計算し、(ほぼ)元の大きさに戻す
		const zoomedRate = croppedImage.getBoundingClientRect().width / croppedImage.clientWidth;
		const widthToRender = croppedSection.getBoundingClientRect().width / zoomedRate;

		const croppedCanvas = await croppedSection?.$toCanvas({ width: widthToRender });
		croppedCanvas?.toBlob(blob => {
			if (!blob) return;
			res(blob);
		});
	});

	const f = await promise;

	emit('ok', f);
	dialogEl.value!.close();
};

const cancel = () => {
	emit('cancel');
	dialogEl.value!.close();
};

const onImageLoad = () => {
	loading.value = false;

	if (cropper) {
		cropper.getCropperImage()!.$center('contain');
		cropper.getCropperSelection()!.$center();
	}
};

onMounted(() => {
	cropper = new Cropper(imgEl.value!, {
	});

	const computedStyle = getComputedStyle(window.document.documentElement);

	const selection = cropper.getCropperSelection()!;
	selection.themeColor = tinycolor(computedStyle.getPropertyValue('--MI_THEME-accent')).toHexString();
	if (props.aspectRatio != null) selection.aspectRatio = props.aspectRatio;
	selection.initialAspectRatio = props.aspectRatio ?? 1;
	selection.outlined = true;

	window.setTimeout(() => {
		cropper!.getCropperImage()!.$center('contain');
		selection.$center();
	}, 100);

	// モーダルオープンアニメーションが終わったあとで再度調整
	window.setTimeout(() => {
		cropper!.getCropperImage()!.$center('contain');
		selection.$center();
	}, 500);
});
</script>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
	transition: opacity 0.5s ease 0.5s;
}
.fade-enter-from,
.fade-leave-to {
	opacity: 0;
}

.mk-cropper-dialog {
	display: flex;
	flex-direction: column;
	width: var(--vw);
	height: var(--vh);
	position: relative;

	> .loading {
		position: absolute;
		z-index: 10;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		-webkit-backdrop-filter: var(--MI-blur, blur(10px));
		backdrop-filter: var(--MI-blur, blur(10px));
		background: rgba(0, 0, 0, 0.5);
	}

	> .container {
		flex: 1;
		width: 100%;
		height: 100%;

		> ::v-deep(cropper-canvas) {
			width: 100%;
			height: 100%;

			> cropper-selection > cropper-handle[action="move"] {
				background: transparent;
			}
		}
	}
}
</style>
