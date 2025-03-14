<template>
    <div :class="cx('container')" role="alert" aria-live="assertive" aria-atomic="true" v-bind="ptm('container')">
        <div :class="cx('content')" v-bind="ptm('content')">
            <template v-if="!templates.message">
                <component :is="templates.icon ? templates.icon : iconComponent.name ? iconComponent : 'span'" :class="cx('icon')" v-bind="ptm('icon')" />
                <div :class="cx('text')" v-bind="ptm('text')">
                    <span :class="cx('summary')" v-bind="ptm('summary')">{{ message.summary }}</span>
                    <div :class="cx('detail')" v-bind="ptm('detail')">{{ message.detail }}</div>
                </div>
            </template>
            <component v-else :is="templates.message" :message="message"></component>
            <div v-if="message.closable !== false" v-bind="ptm('buttonContainer')">
                <button v-ripple :class="cx('button')" type="button" :aria-label="closeAriaLabel" @click="onCloseClick" autofocus v-bind="{ ...closeButtonProps, ...ptm('button') }">
                    <component :is="templates.closeicon || 'TimesIcon'" :class="cx('buttonIcon')" v-bind="ptm('buttonIcon')" />
                </button>
            </div>
        </div>
    </div>
</template>

<script>
import CheckIcon from 'primevue/icons/check';
import ExclamationTriangleIcon from 'primevue/icons/exclamationtriangle';
import InfoCircleIcon from 'primevue/icons/infocircle';
import TimesIcon from 'primevue/icons/times';
import TimesCircleIcon from 'primevue/icons/timescircle';
import Ripple from 'primevue/ripple';
import BaseToast from './BaseToast.vue';

export default {
    name: 'ToastMessage',
    extends: BaseToast,
    emits: ['close'],
    closeTimeout: null,
    props: {
        message: {
            type: null,
            default: null
        },
        templates: {
            type: Object,
            default: null
        },
        closeIcon: {
            type: String,
            default: null
        },
        infoIcon: {
            type: String,
            default: null
        },
        warnIcon: {
            type: String,
            default: null
        },
        errorIcon: {
            type: String,
            default: null
        },
        successIcon: {
            type: String,
            default: null
        },
        closeButtonProps: {
            type: null,
            default: null
        }
    },
    mounted() {
        if (this.message.life) {
            this.closeTimeout = setTimeout(() => {
                this.close({ message: this.message, type: 'life-end' });
            }, this.message.life);
        }
    },
    beforeUnmount() {
        this.clearCloseTimeout();
    },
    methods: {
        close(params) {
            this.$emit('close', params);
        },
        onCloseClick() {
            this.clearCloseTimeout();
            this.close({ message: this.message, type: 'close' });
        },
        clearCloseTimeout() {
            if (this.closeTimeout) {
                clearTimeout(this.closeTimeout);
                this.closeTimeout = null;
            }
        }
    },
    computed: {
        iconComponent() {
            return {
                info: !this.infoIcon && InfoCircleIcon,
                success: !this.successIcon && CheckIcon,
                warn: !this.warnIcon && ExclamationTriangleIcon,
                error: !this.errorIcon && TimesCircleIcon
            }[this.message.severity];
        },
        closeAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.close : undefined;
        }
    },
    components: {
        TimesIcon: TimesIcon,
        InfoCircleIcon: InfoCircleIcon,
        CheckIcon: CheckIcon,
        ExclamationTriangleIcon: ExclamationTriangleIcon,
        TimesCircleIcon: TimesCircleIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
