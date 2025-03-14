<template>
    <div ref="container" :class="cx('root')" :style="sx('root')" @click="onClick" v-bind="ptm('root')" data-pc-name="treeselect">
        <div :class="cx('hiddenInputWrapper')" :style="sx('hiddenAccessible', isUnstyled)" v-bind="ptm('hiddenInputWrapper')" :data-p-hidden-accessible="true">
            <input
                ref="focusInput"
                :id="inputId"
                type="text"
                role="combobox"
                :class="inputClass"
                :style="inputStyle"
                readonly
                :disabled="disabled"
                :tabindex="!disabled ? tabindex : -1"
                :aria-labelledby="ariaLabelledby"
                :aria-label="ariaLabel"
                aria-haspopup="tree"
                :aria-expanded="overlayVisible"
                :aria-controls="listId"
                @focus="onFocus($event)"
                @blur="onBlur($event)"
                @keydown="onKeyDown($event)"
                v-bind="{ ...inputProps, ...ptm('hiddenInput') }"
            />
        </div>
        <div :class="cx('labelContainer')" v-bind="ptm('labelContainer')">
            <div :class="cx('label')" v-bind="ptm('label')">
                <slot name="value" :value="selectedNodes" :placeholder="placeholder">
                    <template v-if="display === 'comma'">
                        {{ label || 'empty' }}
                    </template>
                    <template v-else-if="display === 'chip'">
                        <div v-for="node of selectedNodes" :key="node.key" :class="cx('token')" v-bind="ptm('token')">
                            <span :class="cx('tokenLabel')" v-bind="ptm('tokenLabel')">{{ node.label }}</span>
                        </div>
                        <template v-if="emptyValue">{{ placeholder || 'empty' }}</template>
                    </template>
                </slot>
            </div>
        </div>
        <div :class="cx('trigger')" role="button" aria-haspopup="tree" :aria-expanded="overlayVisible" v-bind="ptm('trigger')">
            <slot name="triggericon" :class="cx('triggerIcon')">
                <component :is="'ChevronDownIcon'" :class="cx('triggerIcon')" v-bind="ptm('triggerIcon')" />
            </slot>
        </div>
        <Portal :appendTo="appendTo">
            <transition name="p-connected-overlay" @enter="onOverlayEnter" @leave="onOverlayLeave" @after-leave="onOverlayAfterLeave">
                <div v-if="overlayVisible" :ref="overlayRef" @click="onOverlayClick" :class="[cx('panel'), panelClass]" @keydown="onOverlayKeydown" v-bind="{ ...panelProps, ...ptm('panel') }">
                    <slot name="header" :value="modelValue" :options="options"></slot>
                    <div :class="cx('panel')" :style="{ 'max-height': scrollHeight }" v-bind="ptm('wrapper')">
                        <TSTree
                            ref="tree"
                            :id="listId"
                            :value="options"
                            :selectionMode="selectionMode"
                            @update:selectionKeys="onSelectionChange"
                            :selectionKeys="modelValue"
                            :expandedKeys="expandedKeys"
                            @update:expandedKeys="onNodeToggle"
                            :metaKeySelection="metaKeySelection"
                            @node-expand="$emit('node-expand', $event)"
                            @node-collapse="$emit('node-collapse', $event)"
                            @node-select="onNodeSelect"
                            @node-unselect="onNodeUnselect"
                            :level="0"
                            :pt="ptm('tree')"
                            data-pc-section="tree"
                        >
                            <template v-if="$slots.itemtogglericon" #togglericon="iconProps">
                                <slot name="itemtogglericon" :node="iconProps.node" :expanded="iconProps.expanded" :class="iconProps.class" />
                            </template>
                            <template v-if="$slots.itemcheckboxicon" #checkboxicon="iconProps">
                                <slot name="itemcheckboxicon" :checked="iconProps.checked" :partialChecked="iconProps.partialChecked" :class="iconProps.class" />
                            </template>
                        </TSTree>
                        <div v-if="emptyOptions" :class="cx('emptyMessage')" v-bind="ptm('emptyMessage')">
                            <slot name="empty">{{ emptyMessageText }}</slot>
                        </div>
                    </div>
                    <slot name="footer" :value="modelValue" :options="options"></slot>
                </div>
            </transition>
        </Portal>
    </div>
</template>

<script>
import ChevronDownIcon from 'primevue/icons/chevrondown';
import OverlayEventBus from 'primevue/overlayeventbus';
import Portal from 'primevue/portal';
import Ripple from 'primevue/ripple';
import Tree from 'primevue/tree';
import { ConnectedOverlayScrollHandler, DomHandler, UniqueComponentId, ZIndexUtils } from 'primevue/utils';
import BaseTreeSelect from './BaseTreeSelect.vue';

export default {
    name: 'TreeSelect',
    extends: BaseTreeSelect,
    emits: ['update:modelValue', 'before-show', 'before-hide', 'change', 'show', 'hide', 'node-select', 'node-unselect', 'node-expand', 'node-collapse', 'focus', 'blur'],
    data() {
        return {
            focused: false,
            overlayVisible: false,
            expandedKeys: {}
        };
    },
    watch: {
        modelValue: {
            handler: function () {
                if (!this.selfChange) {
                    this.updateTreeState();
                }

                this.selfChange = false;
            },
            immediate: true
        },
        options() {
            this.updateTreeState();
        }
    },
    outsideClickListener: null,
    resizeListener: null,
    scrollHandler: null,
    overlay: null,
    selfChange: false,
    selfClick: false,
    beforeUnmount() {
        this.unbindOutsideClickListener();
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
    mounted() {
        this.updateTreeState();
    },
    methods: {
        show() {
            this.$emit('before-show');
            this.overlayVisible = true;
        },
        hide() {
            this.$emit('before-hide');
            this.overlayVisible = false;
            this.$refs.focusInput.focus();
        },
        onFocus(event) {
            this.focused = true;
            this.$emit('focus', event);
        },
        onBlur(event) {
            this.focused = false;
            this.$emit('blur', event);
        },
        onClick(event) {
            if (!this.disabled && (!this.overlay || !this.overlay.contains(event.target))) {
                if (this.overlayVisible) this.hide();
                else this.show();

                this.$refs.focusInput.focus();
            }
        },
        onSelectionChange(keys) {
            this.selfChange = true;
            this.$emit('update:modelValue', keys);
            this.$emit('change', keys);
        },
        onNodeSelect(node) {
            this.$emit('node-select', node);

            if (this.selectionMode === 'single') {
                this.hide();
            }
        },
        onNodeUnselect(node) {
            this.$emit('node-unselect', node);
        },
        onNodeToggle(keys) {
            this.expandedKeys = keys;
        },
        onKeyDown(event) {
            switch (event.code) {
                case 'ArrowDown':
                    this.onArrowDownKey(event);
                    break;

                case 'Space':
                case 'Enter':
                    this.onEnterKey(event);
                    break;

                case 'Escape':
                    this.onEscapeKey(event);

                    break;

                default:
                    break;
            }
        },
        onArrowDownKey(event) {
            if (this.overlayVisible) return;

            this.show();

            this.$nextTick(() => {
                const treeNodeEl = DomHandler.find(this.$refs.tree.$el, '[data-pc-section="treeitem"]');
                const focusedElement = [...treeNodeEl].find((item) => item.getAttribute('tabindex') === '0');

                DomHandler.focus(focusedElement);
            });

            event.preventDefault();
        },
        onEnterKey(event) {
            if (this.overlayVisible) {
                this.hide();
            } else {
                this.onArrowDownKey(event);
            }

            event.preventDefault();
        },
        onEscapeKey(event) {
            if (this.overlayVisible) {
                this.hide();
                event.preventDefault();
            }
        },
        onOverlayEnter(el) {
            ZIndexUtils.set('overlay', el, this.$primevue.config.zIndex.overlay);

            DomHandler.addStyles(el, { position: 'absolute', top: '0', left: '0' });
            this.alignOverlay();
            this.bindOutsideClickListener();
            this.bindScrollListener();
            this.bindResizeListener();
            this.scrollValueInView();
            this.$emit('show');
        },
        onOverlayLeave() {
            this.unbindOutsideClickListener();
            this.unbindScrollListener();
            this.unbindResizeListener();
            this.$emit('hide');
            this.overlay = null;
        },
        onOverlayAfterLeave(el) {
            ZIndexUtils.clear(el);
        },
        alignOverlay() {
            if (this.appendTo === 'self') {
                DomHandler.relativePosition(this.overlay, this.$el);
            } else {
                this.overlay.style.minWidth = DomHandler.getOuterWidth(this.$el) + 'px';
                DomHandler.absolutePosition(this.overlay, this.$el);
            }
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    if (this.overlayVisible && !this.selfClick && this.isOutsideClicked(event)) {
                        this.hide();
                    }

                    this.selfClick = false;
                };

                document.addEventListener('click', this.outsideClickListener);
            }
        },
        unbindOutsideClickListener() {
            if (this.outsideClickListener) {
                document.removeEventListener('click', this.outsideClickListener);
                this.outsideClickListener = null;
            }
        },
        bindScrollListener() {
            if (!this.scrollHandler) {
                this.scrollHandler = new ConnectedOverlayScrollHandler(this.$refs.container, () => {
                    if (this.overlayVisible) {
                        this.hide();
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
                        this.hide();
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
        isOutsideClicked(event) {
            return !(this.$el.isSameNode(event.target) || this.$el.contains(event.target) || (this.overlay && this.overlay.contains(event.target)));
        },
        overlayRef(el) {
            this.overlay = el;
        },
        onOverlayClick(event) {
            OverlayEventBus.emit('overlay-click', {
                originalEvent: event,
                target: this.$el
            });

            this.selfClick = true;
        },
        onOverlayKeydown(event) {
            if (event.code === 'Escape') this.hide();
        },
        findSelectedNodes(node, keys, selectedNodes) {
            if (node) {
                if (this.isSelected(node, keys)) {
                    selectedNodes.push(node);
                    delete keys[node.key];
                }

                if (Object.keys(keys).length && node.children) {
                    for (let childNode of node.children) {
                        this.findSelectedNodes(childNode, keys, selectedNodes);
                    }
                }
            } else {
                for (let childNode of this.options) {
                    this.findSelectedNodes(childNode, keys, selectedNodes);
                }
            }
        },
        isSelected(node, keys) {
            return this.selectionMode === 'checkbox' ? keys[node.key] && keys[node.key].checked : keys[node.key];
        },
        updateTreeState() {
            let keys = { ...this.modelValue };

            this.expandedKeys = {};

            if (keys && this.options) {
                this.updateTreeBranchState(null, null, keys);
            }
        },
        updateTreeBranchState(node, path, keys) {
            if (node) {
                if (this.isSelected(node, keys)) {
                    this.expandPath(path);
                    delete keys[node.key];
                }

                if (Object.keys(keys).length && node.children) {
                    for (let childNode of node.children) {
                        path.push(node.key);
                        this.updateTreeBranchState(childNode, path, keys);
                    }
                }
            } else {
                for (let childNode of this.options) {
                    this.updateTreeBranchState(childNode, [], keys);
                }
            }
        },
        expandPath(path) {
            if (path.length > 0) {
                for (let key of path) {
                    this.expandedKeys[key] = true;
                }
            }
        },
        scrollValueInView() {
            if (this.overlay) {
                let selectedItem = DomHandler.findSingle(this.overlay, '[data-p-highlight="true"]');

                if (selectedItem) {
                    selectedItem.scrollIntoView({ block: 'nearest', inline: 'start' });
                }
            }
        }
    },
    computed: {
        selectedNodes() {
            let selectedNodes = [];

            if (this.modelValue && this.options) {
                let keys = { ...this.modelValue };

                this.findSelectedNodes(null, keys, selectedNodes);
            }

            return selectedNodes;
        },
        label() {
            let value = this.selectedNodes;

            return value.length ? value.map((node) => node.label).join(', ') : this.placeholder;
        },
        emptyMessageText() {
            return this.emptyMessage || this.$primevue.config.locale.emptyMessage;
        },
        emptyValue() {
            return !this.modelValue || Object.keys(this.modelValue).length === 0;
        },
        emptyOptions() {
            return !this.options || this.options.length === 0;
        },
        listId() {
            return UniqueComponentId() + '_list';
        }
    },
    components: {
        TSTree: Tree,
        Portal: Portal,
        ChevronDownIcon: ChevronDownIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
