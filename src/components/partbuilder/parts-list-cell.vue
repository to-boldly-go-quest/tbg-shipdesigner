<template>
	<td class="cell"
		@click="edit_cell"
		@focus="edit_cell"
		tabindex="0"
		:style="computed_style">

		<span
			class="display-span"
			v-show="!is_editing">{{display_value}}</span>
		
		<input
			v-if="is_editing"
			class="edit-input"
			:style="computed_input_style"
			ref="input"
			type="text"
			@blur="commit_edit"
			@keydown="on_keydown"
			v-focus
			v-model="temp_value"/>
	</td>
</template>

<script>

export default {
	name: 'PartsListCell',
	components: {
	},
	directives: {
		focus: {
			inserted(el) {
				el.focus();
			},
		},
	},
	props: {
		part: {
			type: Object,
		},
		field: {
			type: Object,
		},
	},
	data() {
		return {
			is_editing: false,
			temp_value: null,
		};
	},
	computed: {
		computed_style() {
			return Object.assign({
				'width': this.field.width.toString() + 'px',
				'text-align': this.field.align,
			}, this.computed_font);
		},
		computed_input_style() {
			return {
				'text-align': this.field.align,
			};
		},
		computed_font() {
			let style = this.field.style;
			if (!style && this.field.fixed) {
				style = 'fixed';
			}
			switch (style) {
			case 'fixed':
				return {
					['font-family']: "'Roboto Mono', monospace",
					['font-size']: '12px',
				};
			case 'variable':
			default:
				return {};
			}
		},
		display_value() {
			const v = this.part[this.field.name];
			switch (this.field.edit_type) {
			case 'number': {
				if (v === undefined || v === null || Number.isNaN(v)) {
					return '';
				} else if (typeof(v.valueOf()) === 'number') {
					const f = v.toFixed(this.field.fixed);
					return f.replace(/(\..*?)(0+)$/, (match, p1, p2) => p1 + ' '.repeat(p2.length)).replace(/\. ( *)/, '.0$1');
				} else {
					return v;
				}
			}
			case 'string':
				return v;
			}
		},
		value: {
			get() {
				const v = this.part[this.field.name];
				if (this.field.edit_type === 'number') {
					if (v === undefined || v === null || Number.isNaN(v)) {
						return '';
					} else {
						return Number(v);
					}
				} else {
					return v;
				}
			},
			set(value) {
				const v = this.part[this.field.name];
				if ((v === undefined || v === null) && value === '') {
					return;
				}
				if (this.field.edit_type === 'number') {
					const new_value = Number(value);
					if (Number(v) !== new_value && !Number.isNaN(new_value)) {
						this.$store.commit('edit_part', {
							part: this.part,
							field: this.field.name,
							value: new_value,
						});
					}
				} else {
					if (v !== value) {
						this.$store.commit('edit_part', {
							part: this.part,
							field: this.field.name,
							value: value,
						});
					}
				}
			},
		},
	},
	methods: {
		on_keydown(ev) {
			switch (ev.key) {
			case 'Enter':
				// leave focus and let the app save changes
				this.commit_edit();
				break;
			case 'Escape':
				this.abort_edit();
				break;
			}
		},
		abort_edit() {
			this.temp_value = this.value;
			this.is_editing = false;
		},
		edit_cell(ev) {
			this.is_editing = true;
			this.temp_value = this.value;
		},
		commit_edit(ev) {
			this.is_editing = false;
			this.value = this.temp_value;
		},
	},
};
</script>

<style scoped>

input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
	-webkit-appearance: none;
	margin: 0;
}
input[type="number"] {
	-moz-appearance: textfield;
}

.cell {
	border: 1px solid #eee;
	cursor: cell;
}

.display-span {
	width: 100%;
	white-space: pre;
}

.edit-input {
	padding: 0px;
	border-width: 0;
	width: 100%;
	box-sizing: border-box;
}


</style>

<style>

</style>
