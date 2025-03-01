<script>
import BaseComponent from 'primevue/basecomponent';
import { useStyle } from 'primevue/usestyle';

const styles = `
.p-tieredmenu ul {
    margin: 0;
    padding: 0;
    list-style: none;
}

.p-tieredmenu .p-submenu-list {
    position: absolute;
    min-width: 100%;
    z-index: 1;
    display: none;
}

.p-tieredmenu .p-menuitem-link {
    cursor: pointer;
    display: flex;
    align-items: center;
    text-decoration: none;
    overflow: hidden;
    position: relative;
}

.p-tieredmenu .p-menuitem-text {
    line-height: 1;
}

.p-tieredmenu .p-menuitem {
    position: relative;
}

.p-tieredmenu .p-menuitem-link .p-submenu-icon {
    margin-left: auto;
}

.p-tieredmenu .p-menuitem-active > .p-submenu-list {
    display: block;
    left: 100%;
    top: 0;
}
`;

const inlineStyles = {
    submenu: ({ context, processedItem }) => ({ display: context.isItemActive(processedItem) ? 'block' : 'none' })
};

const classes = {
    root: ({ instance, props }) => [
        'p-tieredmenu p-component',
        {
            'p-tieredmenu-overlay': props.popup,
            'p-input-filled': instance.$primevue.config.inputStyle === 'filled',
            'p-ripple-disabled': instance.$primevue.config.ripple === false
        }
    ],
    menu: 'p-tieredmenu-root-list',
    menuitem: ({ context, processedItem }) => [
        'p-menuitem',
        {
            'p-menuitem-active p-highlight': context.isItemActive(processedItem),
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
    text: 'p-menuitem-text',
    submenuIcon: 'p-submenu-icon',
    submenu: 'p-submenu-list',
    separator: 'p-menuitem-separator'
};

const { load: loadStyle, unload: unloadStyle } = useStyle(styles, { id: 'primevue_tieredmenu_style', manual: true });

export default {
    name: 'BaseTieredMenu',
    extends: BaseComponent,
    props: {
        popup: {
            type: Boolean,
            default: false
        },
        model: {
            type: Array,
            default: null
        },
        appendTo: {
            type: String,
            default: 'body'
        },
        autoZIndex: {
            type: Boolean,
            default: true
        },
        baseZIndex: {
            type: Number,
            default: 0
        },
        exact: {
            type: Boolean,
            default: true
        },
        disabled: {
            type: Boolean,
            default: false
        },
        tabindex: {
            type: Number,
            default: 0
        },
        'aria-labelledby': {
            type: String,
            default: null
        },
        'aria-label': {
            type: String,
            default: null
        }
    },
    css: {
        classes,
        inlineStyles
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
