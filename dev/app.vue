<template lang="jade">
	div
		.content(:class="{ hasKeyboard: visible }")
			fieldset
				legend Normal layout
				input#text.input(type="text", placeholder="Text input", @focus="show", data-layout="normal")
			fieldset
				legend Normal layout
				input#text.input(type="text", placeholder="Text input", @focus="show", data-layout="normal")

		.PinYin
			span(v-for="(item, index) in PinYin", key="index", v-text="item", @click="clickPinYin(item)")
		vue-touch-keyboard#keyboard(v-if="visible", :layout="layout", :cancel="hide", :accept="accept", :input="input", :next="next", :options="options", :change="change")

</template>

<script>
	import Vue from "vue";
	import VueTouchKeyboard from "../src";

	import {each, isFunction, cloneDeep, merge} from 'lodash';

	Vue.use(VueTouchKeyboard);

	export default {

		data() {
			return {
				visible: false,
				allLayouts: VueTouchKeyboard.layouts,
				//layout: VueTouchKeyboard.layouts["compact"],
				layout: "mini",
				input: null,
				options: {
					useKbEvents: true
				},
				PinYin: ["拼","品","频","聘","拼"],
				currentCaret: 0,
				prevPinYinCaret: 0
			}
		},
		watch: {
			input (input) {
				this.prevPinYinCaret = input.value.length
			}
		},
		methods: {
			hide() {
				this.visible = false;
			},

			accept(text) {
				//alert("Input text: " + text);
				this.hide();
			},

			clickPinYin (value) {
				let text = this.input.value
				text = text.substring(0, this.prevPinYinCaret) + value + text.substring(this.currentCaret);
				this.input.value = text
				this.prevPinYinCaret++
				this.currentCaret++
			},

			next() {
				let inputs = document.querySelectorAll("input");
				let found = false;
				[].forEach.call(inputs, (item, i) => {
					if (!found && item == this.input && i < inputs.length - 1) {
						found = true;
						this.$nextTick(() => {
							inputs[i+1].focus();
						});
					}
				});
				if (!found) {
					this.input.blur();
					this.hide();
				}
			},

			show(e) {
				this.input = e.target;
				this.layout = e.target.dataset.layout;

				if (!this.visible)
					this.visible = true

				this.$nextTick(() => {
					this.input.scrollIntoView();
				});
			},

			change (text, addChar, caret) {
				this.currentCaret = caret.end
				if (this.currentCaret < this.prevPinYinCaret) {
					let x = this.currentCaret
					this.currentCaret = this.prevPinYinCaret
					this.prevPinYinCaret = x
				}
				console.log(`当前光标的位置:${this.currentCaret}, '上一个拼音输入的位置：'${this.prevPinYinCaret}`)
			}
		},

		mounted() {
			window.app = this;
			this.$nextTick(() => {
				this.input = document.querySelector("input#text");
				this.input.focus();
				//this.visible = true;
			});
		}
	}

</script>

<style lang="sass">

	html {
		font-family: "Arial", sans-serif;
		font-size: 16px;
	}

	* {
		-moz-box-sizing: border-box;
		-webkit-box-sizing: border-box;
		box-sizing: border-box;
	}
	.PinYin {
		background-color: #ccc;
		color: #666;
		padding: 10px;
		z-index: 999999999999;
		position: absolute;
		bottom: 300px;
		span {
			padding: 10px;
		}
	}
	.content {
		text-align: center;
		position: absolute;
		left: 0; right: 0; top: 0; bottom: 0;
		overflow: auto;

		&.hasKeyboard {
			bottom: 18em;
		}
	}

	#keyboard {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;

		z-index: 1000;
		width: 100%;
		max-width: 1000px;
		margin: 0 auto;

		padding: 1em;

		background-color: #EEE;
		box-shadow: 0px -3px 10px rgba(black, 0.3);

		border-radius: 10px;
	}

	fieldset {
		display: block;
		width: 300px;
		padding: 15px;
		margin: 15px auto;
		border-style: solid;
		background-color: #fff;
		border-color: #ddd;
		border-width: 1px;
		border-radius: 4px;
	}

	input.input, select#layoutSelector {
		display: block;
		width: 100%;
		height: 34px;
		padding: 6px 12px;
		font-size: 14px;
		line-height: 1.42857143;
		color: #555;
		background-color: #fff;
		background-image: none;
		border: 1px solid #ccc;
		border-radius: 4px;
		box-shadow: inset 0 1px 1px rgba(0,0,0,.075);
		transition: border-color ease-in-out .15s,box-shadow ease-in-out .15s;

		&:focus {
			border-color: #66afe9;
			outline: 0;
			box-shadow: inset 0 1px 1px rgba(0,0,0,.075),0 0 8px rgba(102,175,233,.6);
		}
	}

</style>
