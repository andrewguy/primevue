<template>
    <Portal>
        <div ref="container" :class="cx('root')" v-bind="{ ...$attrs, ...ptm('root') }">
            <transition-group name="p-toast-message" tag="div" @enter="onEnter" @leave="onLeave" v-bind="ptm('message')">
                <ToastMessage
                    v-for="msg of messages"
                    :key="msg.id"
                    :message="msg"
                    :templates="$slots"
                    :closeIcon="closeIcon"
                    :infoIcon="infoIcon"
                    :warnIcon="warnIcon"
                    :errorIcon="errorIcon"
                    :successIcon="successIcon"
                    :closeButtonProps="closeButtonProps"
                    @close="remove($event)"
                    :pt="pt"
                />
            </transition-group>
        </div>
    </Portal>
</template>

<script>
import BaseToast from './BaseToast.vue';
import Portal from 'primevue/portal';
import ToastEventBus from 'primevue/toasteventbus';
import { ObjectUtils, UniqueComponentId, ZIndexUtils } from 'primevue/utils';
import ToastMessage from './ToastMessage.vue';

var messageIdx = 0;

export default {
    name: 'Toast',
    extends: BaseToast,
    inheritAttrs: false,
    emits: ['close', 'life-end'],

    data() {
        return {
            messages: []
        };
    },
    styleElement: null,
    mounted() {
        ToastEventBus.on('add', this.onAdd);
        ToastEventBus.on('remove-group', this.onRemoveGroup);
        ToastEventBus.on('remove-all-groups', this.onRemoveAllGroups);

        if (this.breakpoints) {
            this.createStyle();
        }
    },
    beforeUnmount() {
        this.destroyStyle();

        if (this.$refs.container && this.autoZIndex) {
            ZIndexUtils.clear(this.$refs.container);
        }

        ToastEventBus.off('add', this.onAdd);
        ToastEventBus.off('remove-group', this.onRemoveGroup);
        ToastEventBus.off('remove-all-groups', this.onRemoveAllGroups);
    },
    methods: {
        add(message) {
            if (message.id == null) {
                message.id = messageIdx++;
            }

            this.messages = [...this.messages, message];
        },
        remove(params) {
            let index = -1;

            for (let i = 0; i < this.messages.length; i++) {
                if (this.messages[i] === params.message) {
                    index = i;
                    break;
                }
            }

            this.messages.splice(index, 1);
            this.$emit(params.type, { message: params.message });
        },
        onAdd(message) {
            if (this.group == message.group) {
                this.add(message);
            }
        },
        onRemoveGroup(group) {
            if (this.group === group) {
                this.messages = [];
            }
        },
        onRemoveAllGroups() {
            this.messages = [];
        },
        onEnter() {
            this.$refs.container.setAttribute(this.attributeSelector, '');

            if (this.autoZIndex) {
                ZIndexUtils.set('modal', this.$refs.container, this.baseZIndex || this.$primevue.config.zIndex.modal);
            }
        },
        onLeave() {
            if (this.$refs.container && this.autoZIndex && ObjectUtils.isEmpty(this.messages)) {
                setTimeout(() => {
                    ZIndexUtils.clear(this.$refs.container);
                }, 200);
            }
        },
        createStyle() {
            if (!this.styleElement) {
                this.styleElement = document.createElement('style');
                this.styleElement.type = 'text/css';
                document.head.appendChild(this.styleElement);

                let innerHTML = '';

                for (let breakpoint in this.breakpoints) {
                    let breakpointStyle = '';

                    for (let styleProp in this.breakpoints[breakpoint]) {
                        breakpointStyle += styleProp + ':' + this.breakpoints[breakpoint][styleProp] + '!important;';
                    }

                    innerHTML += `
                        @media screen and (max-width: ${breakpoint}) {
                            .p-toast[${this.attributeSelector}] {
                                ${breakpointStyle}
                            }
                        }
                    `;
                }

                this.styleElement.innerHTML = innerHTML;
            }
        },
        destroyStyle() {
            if (this.styleElement) {
                document.head.removeChild(this.styleElement);
                this.styleElement = null;
            }
        }
    },
    computed: {
        attributeSelector() {
            return UniqueComponentId();
        }
    },
    components: {
        ToastMessage: ToastMessage,
        Portal: Portal
    }
};
</script>
