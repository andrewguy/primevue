<template>
    <div role="progressbar" :class="cx('root')" aria-valuemin="0" :aria-valuenow="value" aria-valuemax="100" v-bind="ptm('root')" data-pc-name="progressbar">
        <div v-if="determinate" :class="cx('value')" :style="progressStyle" v-bind="ptm('value')">
            <div v-if="value != null && value !== 0 && showValue" :class="cx('label')" v-bind="ptm('label')">
                <slot>{{ value + '%' }}</slot>
            </div>
        </div>
        <div v-if="indeterminate" :class="cx('value')" v-bind="ptm('value')">
            <div :class="cx('label')" v-bind="ptm('label')"></div>
        </div>
    </div>
</template>

<script>
import BaseProgressBar from './BaseProgressBar.vue';

export default {
    name: 'ProgressBar',
    extends: BaseProgressBar,
    computed: {
        progressStyle() {
            return {
                width: this.value + '%',
                display: 'flex'
            };
        },
        indeterminate() {
            return this.mode === 'indeterminate';
        },
        determinate() {
            return this.mode === 'determinate';
        }
    }
};
</script>
