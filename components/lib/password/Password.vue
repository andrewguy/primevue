<template>
    <div :class="cx('root')" :style="sx('root')" v-bind="ptm('root')" data-pc-name="password">
        <PInputText
            ref="input"
            :id="inputId"
            :type="inputType"
            :class="[cx('input'), inputClass]"
            :style="inputStyle"
            :value="modelValue"
            :aria-labelledby="ariaLabelledby"
            :aria-label="ariaLabel"
            :aria-controls="(panelProps && panelProps.id) || panelId || panelUniqueId"
            :aria-expanded="overlayVisible"
            :aria-haspopup="true"
            :placeholder="placeholder"
            :required="required"
            @input="onInput"
            @focus="onFocus"
            @blur="onBlur"
            @keyup="onKeyUp"
            @invalid="onInvalid"
            v-bind="{ ...inputProps, ...ptm('input') }"
        />
        <slot v-if="toggleMask && unmasked" name="hideicon" :onClick="onMaskToggle">
            <component :is="hideIcon ? 'i' : 'EyeSlashIcon'" :class="hideIcon" @click="onMaskToggle" v-bind="ptm('hideIcon')" />
        </slot>
        <slot v-if="toggleMask && !unmasked" name="showicon" :onClick="onMaskToggle">
            <component :is="showIcon ? 'i' : 'EyeIcon'" :class="showIcon" @click="onMaskToggle" v-bind="ptm('showIcon')" />
        </slot>
        <span :class="cx('hiddenAccesible')" :style="sx('hiddenAccessible', isUnstyled)" aria-live="polite" v-bind="ptm('hiddenAccesible')" :data-p-hidden-accessible="true">
            {{ infoText }}
        </span>
        <Portal :appendTo="appendTo">
            <transition name="p-connected-overlay" @enter="onOverlayEnter" @leave="onOverlayLeave" @after-leave="onOverlayAfterLeave">
                <div v-if="overlayVisible" :ref="overlayRef" :id="panelId || panelUniqueId" :class="[cx('panel'), panelClass]" :style="panelStyle" @click="onOverlayClick" v-bind="{ ...panelProps, ...ptm('panel') }">
                    <slot name="header"></slot>
                    <slot name="content">
                        <div :class="cx('meter')" v-bind="ptm('meter')">
                            <div :class="cx('meterLabel')" :style="{ width: meter ? meter.width : '' }" v-bind="ptm('meterLabel')"></div>
                        </div>
                        <div :class="cx('info')" v-bind="ptm('info')">{{ infoText }}</div>
                    </slot>
                    <slot name="footer"></slot>
                </div>
            </transition>
        </Portal>
    </div>
</template>

<script>
import EyeIcon from 'primevue/icons/eye';
import EyeSlashIcon from 'primevue/icons/eyeslash';
import InputText from 'primevue/inputtext';
import OverlayEventBus from 'primevue/overlayeventbus';
import Portal from 'primevue/portal';
import { ConnectedOverlayScrollHandler, DomHandler, UniqueComponentId, ZIndexUtils } from 'primevue/utils';
import BasePassword from './BasePassword.vue';

export default {
    name: 'Password',
    extends: BasePassword,
    emits: ['update:modelValue', 'change', 'focus', 'blur', 'invalid'],
    data() {
        return {
            overlayVisible: false,
            meter: null,
            infoText: null,
            focused: false,
            unmasked: false
        };
    },
    mediumCheckRegExp: null,
    strongCheckRegExp: null,
    resizeListener: null,
    scrollHandler: null,
    overlay: null,
    mounted() {
        this.infoText = this.promptText;
        this.mediumCheckRegExp = new RegExp(this.mediumRegex);
        this.strongCheckRegExp = new RegExp(this.strongRegex);
    },
    beforeUnmount() {
        this.unbindResizeListener();

        if (this.scrollHandler) {
            this.scrollHandler.destroy();
            this.scrollHandler = null;
        }

        if (this.overlay) {
            ZIndexUtils.clear(this.overlay);
            this.overlay = null;
        }
    },
    methods: {
        onOverlayEnter(el) {
            ZIndexUtils.set('overlay', el, this.$primevue.config.zIndex.overlay);

            DomHandler.addStyles(el, { position: 'absolute', top: '0', left: '0' });
            this.alignOverlay();
            this.bindScrollListener();
            this.bindResizeListener();
        },
        onOverlayLeave() {
            this.unbindScrollListener();
            this.unbindResizeListener();
            this.overlay = null;
        },
        onOverlayAfterLeave(el) {
            ZIndexUtils.clear(el);
        },
        alignOverlay() {
            if (this.appendTo === 'self') {
                DomHandler.relativePosition(this.overlay, this.$refs.input.$el);
            } else {
                this.overlay.style.minWidth = DomHandler.getOuterWidth(this.$refs.input.$el) + 'px';
                DomHandler.absolutePosition(this.overlay, this.$refs.input.$el);
            }
        },
        testStrength(str) {
            let level = 0;

            if (this.strongCheckRegExp.test(str)) level = 3;
            else if (this.mediumCheckRegExp.test(str)) level = 2;
            else if (str.length) level = 1;

            return level;
        },
        onInput(event) {
            this.$emit('update:modelValue', event.target.value);
        },
        onFocus(event) {
            this.focused = true;

            if (this.feedback) {
                this.setPasswordMeter(this.modelValue);
                this.overlayVisible = true;
            }

            this.$emit('focus', event);
        },
        onBlur(event) {
            this.focused = false;

            if (this.feedback) {
                this.overlayVisible = false;
            }

            this.$emit('blur', event);
        },
        onKeyUp(event) {
            if (this.feedback) {
                const value = event.target.value;
                const { meter, label } = this.checkPasswordStrength(value);

                this.meter = meter;
                this.infoText = label;

                if (event.code === 'Escape') {
                    this.overlayVisible && (this.overlayVisible = false);

                    return;
                }

                if (!this.overlayVisible) {
                    this.overlayVisible = true;
                }
            }
        },
        setPasswordMeter() {
            if (!this.modelValue) return;

            const { meter, label } = this.checkPasswordStrength(this.modelValue);

            this.meter = meter;
            this.infoText = label;

            if (!this.overlayVisible) {
                this.overlayVisible = true;
            }
        },
        checkPasswordStrength(value) {
            let label = null;
            let meter = null;

            switch (this.testStrength(value)) {
                case 1:
                    label = this.weakText;
                    meter = {
                        strength: 'weak',
                        width: '33.33%'
                    };
                    break;

                case 2:
                    label = this.mediumText;
                    meter = {
                        strength: 'medium',
                        width: '66.66%'
                    };
                    break;

                case 3:
                    label = this.strongText;
                    meter = {
                        strength: 'strong',
                        width: '100%'
                    };
                    break;

                default:
                    label = this.promptText;
                    meter = null;
                    break;
            }

            return { label, meter };
        },
        onInvalid(event) {
            this.$emit('invalid', event);
        },
        bindScrollListener() {
            if (!this.scrollHandler) {
                this.scrollHandler = new ConnectedOverlayScrollHandler(this.$refs.input.$el, () => {
                    if (this.overlayVisible) {
                        this.overlayVisible = false;
                    }
                });
            }

            this.scrollHandler.bindScrollListener();
        },
        unbindScrollListener() {
            if (this.scrollHandler) {
                this.scrollHandler.unbindScrollListener();
            }
        },
        bindResizeListener() {
            if (!this.resizeListener) {
                this.resizeListener = () => {
                    if (this.overlayVisible && !DomHandler.isTouchDevice()) {
                        this.overlayVisible = false;
                    }
                };

                window.addEventListener('resize', this.resizeListener);
            }
        },
        unbindResizeListener() {
            if (this.resizeListener) {
                window.removeEventListener('resize', this.resizeListener);
                this.resizeListener = null;
            }
        },
        overlayRef(el) {
            this.overlay = el;
        },
        onMaskToggle() {
            this.unmasked = !this.unmasked;
        },
        onOverlayClick(event) {
            OverlayEventBus.emit('overlay-click', {
                originalEvent: event,
                target: this.$el
            });
        }
    },
    computed: {
        inputType() {
            return this.unmasked ? 'text' : 'password';
        },
        filled() {
            return this.modelValue != null && this.modelValue.toString().length > 0;
        },
        weakText() {
            return this.weakLabel || this.$primevue.config.locale.weak;
        },
        mediumText() {
            return this.mediumLabel || this.$primevue.config.locale.medium;
        },
        strongText() {
            return this.strongLabel || this.$primevue.config.locale.strong;
        },
        promptText() {
            return this.promptLabel || this.$primevue.config.locale.passwordPrompt;
        },
        panelUniqueId() {
            return UniqueComponentId() + '_panel';
        }
    },
    components: {
        PInputText: InputText,
        Portal: Portal,
        EyeSlashIcon: EyeSlashIcon,
        EyeIcon: EyeIcon
    }
};
</script>
