<template>
<XModalWindow ref="dialog"
	:width="400"
	:with-ok-button="true"
	:ok-button-disabled="!funcName || !body"
	@ok="ok()"
	@close="$refs.dialog.close()"
	@closed="$emit('closed')"
>
	<template #header>{{ $ts._mfmpad.functionsBuilder }}</template>
	<section class="_section">
		<div class="_content">
			<MkSelect v-model:value="funcName">
				<template #label>{{ $ts._mfmpad._functions.kind }}</template>
				<option v-for="(fn, name) in funcs" :value="name" :key="name" v-text="$ts._mfm[`${name}`]"/>
			</MkSelect>
		</div>
	</section>
	<MkInfo warn v-if="isGroundpolisExtension">
		{{ $ts._mfmpad._functions.warnGroundpolisExtension }}
	</MkInfo>
	<section class="_section" v-if="args.length > 0">
		
		<div class="_content">
			<template v-for="arg in args" :key="arg.key">
				<MkSwitch v-model:value="arg.enabled">
					{{ getProp(arg.key).label || arg.key }}
					<template #desc>{{ getProp(arg.key).description }}</template>
				</MkSwitch>
				<MkInput v-model:value="arg.value" v-if="getProp(arg.key).hasValue && arg.enabled"><span v-text="$ts.input"/></MkInput>				
			</template>
		</div>
	</section>
	<section class="_section" v-if="funcName">
		<div class="_content">
			<MkInput v-model:value="body"><span v-text="$ts._mfmpad._functions.text"/></MkInput>
		</div>
	</section>
	<section class="_section" v-if="gpfm">
		<div class="_content">
			<div v-text="$ts.preview" />
			<Mfm :text="gpfm" />
		</div>
	</section>
</XModalWindow>
</template>

<script lang="ts">
import { computed, defineComponent, markRaw, ref, SetupContext, watchEffect } from 'vue';
import XModalWindow from '@/components/ui/modal-window.vue';
import MkSelect from './ui/select.vue';
import MkSwitch from './ui/switch.vue';
import MkInput from './ui/input.vue';
import MkInfo from './ui/info.vue';
import { mfmFunctions } from './mfm.functions';

export default defineComponent({
	components: {
		XModalWindow,
		MkSelect,
		MkSwitch,
		MkInput,
		MkInfo,
	},

	setup(props: {}, ctx: SetupContext) {
		const funcName = ref<string>('');
		const funcs  = markRaw(mfmFunctions);
		const func = computed(() => funcs[funcName.value]);
		const isGroundpolisExtension = computed(() => func.value && typeof func.value === 'object' && func.value.isGroundpolisExtension);
		const args = ref<{ key: string, value?: string, enabled: boolean }[]>([]);
		const body = ref<string>('');
		const gpfm = computed(() => {
			if (!funcName.value || !body.value) return null;
			const arg = args.value
					.filter(a => a.enabled)
					.map(a => a.value ? `${a.key}=${a.value}` : a.key)
					.join(',');

			return `$[${funcName.value}${arg.length > 0 ? '.' : ''}${arg} ${body.value}]`
		});

		watchEffect(() => {
			args.value = [];

			if (func.value && typeof func.value !== 'string' && func.value.props) {
				for (const key in func.value.props) {
					args.value.push({ key, enabled: false });
				}
			}
		});

		return {
			funcs,
			funcName,
			args,
			body,
			gpfm,
			isGroundpolisExtension,

			getProp(name: string) {
				if (func.value && typeof func.value !== 'string' && func.value.props && func.value.props[name]) {
					return func.value.props[name];
				}
				return {};
			},

			ok() {
				
				ctx.emit('done', gpfm.value);
				this.$refs.dialog.close();
			},
		};
	},
	// components: {
	// 	XModalWindow,
	// 	MkSwitch,
	// 	MkInfo,
	// 	MkButton
	// },

	// props: {
	// },

	// emits: ['done', 'closed'],

	// data() {
	// 	return {
	// 		funcs,
	// 	};
	// },

	// computed: {
	// },

	// created() {
	// },

	// methods: {
	// 	ok() {
	// 		const includingTypes = this.useGlobalSetting ? null : (Object.keys(this.typesMap) as typeof notificationTypes[number][])
	// 			.filter(type => this.typesMap[type]);

	// 		this.$emit('done', { includingTypes });
	// 		this.$refs.dialog.close();
	// 	},
	// }
});
</script>
