<template>
    <input :class="cx('root')" :value="modelValue" @input="onInput" v-bind="ptm('root')" data-pc-name="inputtext" />
</template>

<script>
import BaseInputText from './BaseInputText.vue';

export default {
    name: 'InputText',
    extends: BaseInputText,
    emits: ['update:modelValue'],
    methods: {
        onInput(event) {
            this.$emit('update:modelValue', event.target.value);
        }
    },
    computed: {
        filled() {
            return this.modelValue != null && this.modelValue.toString().length > 0;
        }
    }
};
</script>
