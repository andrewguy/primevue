<script>
import BaseComponent from 'primevue/basecomponent';
import { useStyle } from 'primevue/usestyle';

const styles = `
.p-colorpicker-panel .p-colorpicker-color {
    background: transparent url("./images/color.png") no-repeat left top;
}

.p-colorpicker-panel .p-colorpicker-hue {
    background: transparent url("./images/hue.png") no-repeat left top;
}
`;

const classes = {
    root: ({ props }) => ['p-colorpicker p-component', { 'p-colorpicker-overlay': !props.inline }],
    input: ({ props }) => ['p-colorpicker-preview p-inputtext', { 'p-disabled': props.disabled }],
    panel: ({ instance, props }) => [
        'p-colorpicker-panel',
        {
            'p-colorpicker-overlay-panel': !props.inline,
            'p-disabled': props.disabled,
            'p-input-filled': instance.$primevue.config.inputStyle === 'filled',
            'p-ripple-disabled': instance.$primevue.config.ripple === false
        }
    ],
    content: 'p-colorpicker-content',
    selector: 'p-colorpicker-color-selector',
    color: 'p-colorpicker-color',
    colorHandle: 'p-colorpicker-color-handle',
    hue: 'p-colorpicker-hue',
    hueHandle: 'p-colorpicker-hue-handle'
};

const { load: loadStyle, unload: unloadStyle } = useStyle(styles, { id: 'primevue_colorpicker_style', manual: true });

export default {
    name: 'BaseColorPicker',
    extends: BaseComponent,
    props: {
        modelValue: {
            type: null,
            default: null
        },
        defaultColor: {
            type: null,
            default: 'ff0000'
        },
        inline: {
            type: Boolean,
            default: false
        },
        format: {
            type: String,
            default: 'hex'
        },
        disabled: {
            type: Boolean,
            default: false
        },
        tabindex: {
            type: String,
            default: null
        },
        autoZIndex: {
            type: Boolean,
            default: true
        },
        baseZIndex: {
            type: Number,
            default: 0
        },
        appendTo: {
            type: String,
            default: 'body'
        },
        panelClass: null
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
