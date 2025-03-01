<script>
import BaseComponent from 'primevue/basecomponent';
import { useStyle } from 'primevue/usestyle';

const styles = `
.p-panelmenu .p-panelmenu-header-action {
    display: flex;
    align-items: center;
    user-select: none;
    cursor: pointer;
    position: relative;
    text-decoration: none;
}

.p-panelmenu .p-panelmenu-header-action:focus {
    z-index: 1;
}

.p-panelmenu .p-submenu-list {
    margin: 0;
    padding: 0;
    list-style: none;
}

.p-panelmenu .p-menuitem-link {
    display: flex;
    align-items: center;
    user-select: none;
    cursor: pointer;
    text-decoration: none;
    position: relative;
    overflow: hidden;
}

.p-panelmenu .p-menuitem-text {
    line-height: 1;
}
`;

const classes = {
    root: 'p-panelmenu p-component',
    panel: 'p-panelmenu-panel',
    header: ({ instance, item }) => [
        'p-panelmenu-header',
        {
            'p-highlight': instance.isItemActive(item),
            'p-disabled': instance.isItemDisabled(item)
        }
    ],
    headerContent: 'p-panelmenu-header-content',
    headerAction: ({ instance, isActive, isExactActive }) => [
        'p-panelmenu-header-action',
        {
            'router-link-active': isActive,
            'router-link-active-exact': instance.exact && isExactActive
        }
    ],
    headerIcon: 'p-menuitem-icon',
    headerLabel: 'p-menuitem-text',
    toggleableContent: 'p-toggleable-content',
    menuContent: 'p-panelmenu-content',
    menu: 'p-panelmenu-root-list',
    menuitem: ({ context, processedItem }) => [
        'p-menuitem',
        {
            'p-focus': context.isItemFocused(processedItem),
            'p-disabled': context.isItemDisabled(processedItem)
        }
    ],
    content: 'p-menuitem-content',
    action: ({ context, isActive, isExactActive }) => [
        'p-menuitem-link',
        {
            'router-link-active': isActive,
            'router-link-active-exact': context.exact && isExactActive
        }
    ],
    icon: 'p-menuitem-icon',
    label: 'p-menuitem-text',
    submenuIcon: 'p-submenu-icon',
    submenu: 'p-submenu-list',
    separator: 'p-menuitem-separator'
};

const { load: loadStyle, unload: unloadStyle } = useStyle(styles, { id: 'primevue_panelmenu_style', manual: true });

export default {
    name: 'BasePanelMenu',
    extends: BaseComponent,
    props: {
        model: {
            type: Array,
            default: null
        },
        expandedKeys: {
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
    css: {
        classes
    },
    watch: {
        isUnstyled: {
            immediate: true,
            handler(newValue) {
                !newValue && loadStyle();
            }
        }
    }
};
</script>
