<template>
    <ul :class="level === 0 ? getCXOptions('menu') : getCXOptions('submenu')" :tabindex="tabindex" v-bind="level === 0 ? ptm('menu') : ptm('submenu')">
        <li v-if="submenu" :class="[getCXOptions('submenuHeader', { submenu }), getItemProp(submenu, 'class')]" :style="getItemProp(submenu, 'style')" role="presentation" v-bind="ptm('submenuHeader')">{{ getItemLabel(submenu) }}</li>
        <template v-for="(processedItem, index) of items" :key="getItemKey(processedItem)">
            <li
                v-if="isItemVisible(processedItem) && !getItemProp(processedItem, 'separator')"
                :id="getItemId(processedItem)"
                :style="getItemProp(processedItem, 'style')"
                :class="[getCXOptions('menuitem', { processedItem }), getItemProp(processedItem, 'class')]"
                role="menuitem"
                :aria-label="getItemLabel(processedItem)"
                :aria-disabled="isItemDisabled(processedItem) || undefined"
                :aria-expanded="isItemGroup(processedItem) ? isItemActive(processedItem) : undefined"
                :aria-haspopup="isItemGroup(processedItem) && !getItemProp(processedItem, 'to') ? 'menu' : undefined"
                :aria-level="level + 1"
                :aria-setsize="getAriaSetSize()"
                :aria-posinset="getAriaPosInset(index)"
                v-bind="getPTOptions(processedItem, 'menuitem')"
                :data-p-highlight="isItemActive(processedItem)"
                :data-p-focused="isItemFocused(processedItem)"
                :data-p-disabled="isItemDisabled(processedItem)"
            >
                <div :class="getCXOptions('content')" @click="onItemClick($event, processedItem)" @mouseenter="onItemMouseEnter($event, processedItem)" v-bind="getPTOptions(processedItem, 'content')">
                    <template v-if="!templates.item">
                        <router-link v-if="getItemProp(processedItem, 'to') && !isItemDisabled(processedItem)" v-slot="{ navigate, href, isActive, isExactActive }" :to="getItemProp(processedItem, 'to')" custom>
                            <a v-ripple :href="href" :class="getCXOptions('action', { isActive, isExactActive })" tabindex="-1" aria-hidden="true" @click="onItemActionClick($event, navigate)" v-bind="getPTOptions(processedItem, 'action')">
                                <component v-if="templates.itemicon" :is="templates.itemicon" :item="processedItem.item" :class="[getCXOptions('icon'), getItemProp(processedItem, 'icon')]" />
                                <span v-else-if="getItemProp(processedItem, 'icon')" :class="[getCXOptions('icon'), getItemProp(processedItem, 'icon')]" v-bind="getPTOptions(processedItem, 'icon')" />
                                <span :class="getCXOptions('text')" v-bind="getPTOptions(processedItem, 'label')">{{ getItemLabel(processedItem) }}</span>
                            </a>
                        </router-link>
                        <a v-else v-ripple :href="getItemProp(processedItem, 'url')" :class="getCXOptions('action')" :target="getItemProp(processedItem, 'target')" tabindex="-1" aria-hidden="true" v-bind="getPTOptions(processedItem, 'action')">
                            <component v-if="templates.itemicon" :is="templates.itemicon" :item="processedItem.item" :class="[getCXOptions('icon'), getItemProp(processedItem, 'icon')]" />
                            <span v-else-if="getItemProp(processedItem, 'icon')" :class="[getCXOptions('icon'), getItemProp(processedItem, 'icon')]" v-bind="getPTOptions(processedItem, 'icon')" />
                            <span :class="getCXOptions('text')" v-bind="getPTOptions(processedItem, 'label')">{{ getItemLabel(processedItem) }}</span>
                            <template v-if="isItemGroup(processedItem)">
                                <component v-if="templates.submenuicon" :is="templates.submenuicon" :active="isItemActive(processedItem)" :class="getCXOptions('submenuIcon')" v-bind="getPTOptions(processedItem, 'submenuIcon')" />
                                <component v-else :is="horizontal ? 'AngleDownIcon' : 'AngleRightIcon'" :class="getCXOptions('submenuIcon')" v-bind="getPTOptions(processedItem, 'submenuIcon')" />
                            </template>
                        </a>
                    </template>
                    <component v-else :is="templates.item" :item="processedItem.item"></component>
                </div>
                <div v-if="isItemVisible(processedItem) && isItemGroup(processedItem)" :class="getCXOptions('panel')" v-bind="ptm('panel')">
                    <div :class="getCXOptions('grid')" v-bind="ptm('grid')">
                        <div v-for="col of processedItem.items" :key="getItemKey(col)" :class="getCXOptions('column', { processedItem })" v-bind="ptm('column')">
                            <MegaMenuSub
                                v-for="submenu of col"
                                :key="getSubListKey(submenu)"
                                :id="getSubListId(submenu)"
                                :style="getSXOptions('submenu', { processedItem })"
                                role="menu"
                                :menuId="menuId"
                                :focusedItemId="focusedItemId"
                                :submenu="submenu"
                                :items="submenu.items"
                                :templates="templates"
                                :exact="exact"
                                :level="level + 1"
                                :pt="pt"
                                @item-click="$emit('item-click', $event)"
                                @item-mouseenter="$emit('item-mouseenter', $event)"
                            />
                        </div>
                    </div>
                </div>
            </li>
            <li
                v-if="isItemVisible(processedItem) && getItemProp(processedItem, 'separator')"
                :id="getItemId(processedItem)"
                :class="[getCXOptions('separator'), getItemProp(processedItem, 'class')]"
                :style="getItemProp(processedItem, 'style')"
                role="separator"
                v-bind="ptm('separator')"
            ></li>
        </template>
    </ul>
</template>

<script>
import AngleDownIcon from 'primevue/icons/angledown';
import AngleRightIcon from 'primevue/icons/angleright';
import Ripple from 'primevue/ripple';
import { ObjectUtils } from 'primevue/utils';
import BaseMegaMenu from './BaseMegaMenu.vue';

export default {
    name: 'MegaMenuSub',
    extends: BaseMegaMenu,
    emits: ['item-click', 'item-mouseenter'],
    props: {
        menuId: {
            type: String,
            default: null
        },
        focusedItemId: {
            type: String,
            default: null
        },
        horizontal: {
            type: Boolean,
            default: false
        },
        submenu: {
            type: Object,
            default: null
        },
        items: {
            type: Array,
            default: null
        },
        level: {
            type: Number,
            default: 0
        },
        templates: {
            type: Object,
            default: null
        },
        activeItem: {
            type: Object,
            default: null
        },
        exact: {
            type: Boolean,
            default: true
        },
        tabindex: {
            type: Number,
            default: 0
        }
    },
    methods: {
        getSubListId(processedItem) {
            return `${this.getItemId(processedItem)}_list`;
        },
        getSubListKey(processedItem) {
            return this.getSubListId(processedItem);
        },
        getItemId(processedItem) {
            return `${this.menuId}_${processedItem.key}`;
        },
        getItemKey(processedItem) {
            return this.getItemId(processedItem);
        },
        getItemProp(processedItem, name, params) {
            return processedItem && processedItem.item ? ObjectUtils.getItemValue(processedItem.item[name], params) : undefined;
        },
        getItemLabel(processedItem) {
            return this.getItemProp(processedItem, 'label');
        },
        getPTOptions(processedItem, key) {
            return this.ptm(key, {
                context: {
                    active: this.isItemActive(processedItem),
                    focused: this.isItemFocused(processedItem)
                }
            });
        },
        getCXOptions(key, params) {
            return this.cx(key, {
                ...params,
                context: this
            });
        },
        getSXOptions(key, params) {
            return this.sx(key, true, {
                ...params,
                context: this
            });
        },
        isItemActive(processedItem) {
            return ObjectUtils.isNotEmpty(this.activeItem) ? this.activeItem.key === processedItem.key : false;
        },
        isItemVisible(processedItem) {
            return this.getItemProp(processedItem, 'visible') !== false;
        },
        isItemDisabled(processedItem) {
            return this.getItemProp(processedItem, 'disabled');
        },
        isItemFocused(processedItem) {
            return this.focusedItemId === this.getItemId(processedItem);
        },
        isItemGroup(processedItem) {
            return ObjectUtils.isNotEmpty(processedItem.items);
        },
        onItemClick(event, processedItem) {
            this.getItemProp(processedItem, 'command', { originalEvent: event, item: processedItem.item });
            this.$emit('item-click', { originalEvent: event, processedItem, isFocus: true });
        },
        onItemMouseEnter(event, processedItem) {
            this.$emit('item-mouseenter', { originalEvent: event, processedItem });
        },
        onItemActionClick(event, navigate) {
            navigate && navigate(event);
        },
        getAriaSetSize() {
            return this.items.filter((processedItem) => this.isItemVisible(processedItem) && !this.getItemProp(processedItem, 'separator')).length;
        },
        getAriaPosInset(index) {
            return index - this.items.slice(0, index).filter((processedItem) => this.isItemVisible(processedItem) && this.getItemProp(processedItem, 'separator')).length + 1;
        }
    },
    components: {
        AngleRightIcon: AngleRightIcon,
        AngleDownIcon: AngleDownIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
