<template>
	<img
		v-if="imageThumbnail && !imgError"
		:src="imageThumbnail"
		:class="{ 'is-svg': value && value.type?.includes('svg') }"
		:alt="value.title"
		@error="imgError = true"
	/>
	<div v-else ref="previewEl" class="preview">
		<span v-if="fileExtension" class="extension">
			{{ fileExtension }}
		</span>

		<v-icon v-else name="folder_open" />
	</div>
</template>

<script lang="ts">
import { defineComponent, PropType, computed, ref } from 'vue';
import readableMimeType from '@/utils/readable-mime-type';
import useElementSize from '@/composables/use-element-size';
import { getRootPath } from '@/utils/get-root-path';
import { addTokenToURL } from '@/api';

type File = {
	id: string;
	type: string;
	title: string;
};

export default defineComponent({
	props: {
		value: {
			type: Object as PropType<File>,
			default: null,
		},
	},
	setup(props) {
		const previewEl = ref<Element>();
		const imgError = ref(false);

		const fileExtension = computed(() => {
			if (!props.value) return null;
			return readableMimeType(props.value.type, true);
		});

		const imageThumbnail = computed(() => {
			if (!props.value) return null;
			if (props.value.type?.includes('svg')) return addTokenToURL(getRootPath() + `assets/${props.value.id}`);
			if (props.value.type?.includes('image') === false) return null;
			return addTokenToURL(getRootPath() + `assets/${props.value.id}?key=system-small-cover`);
		});

		const { height } = useElementSize(previewEl);

		return { fileExtension, imageThumbnail, previewEl, height, imgError };
	},
});
</script>

<style lang="scss" scoped>
img {
	height: 100%;
	object-fit: cover;
	border-radius: var(--border-radius);
	aspect-ratio: 1;
}

.preview {
	--v-icon-color: var(--foreground-subdued);

	position: relative;
	display: inline-flex;
	align-items: center;
	justify-content: center;
	height: 100%;
	overflow: hidden;
	background-color: var(--background-normal);
	border-radius: var(--border-radius);
	aspect-ratio: 1;

	&.has-file {
		background-color: var(--primary-alt);
	}
}

.extension {
	color: var(--primary);
	font-weight: 600;
	font-size: 11px;
	text-transform: uppercase;
}
</style>
