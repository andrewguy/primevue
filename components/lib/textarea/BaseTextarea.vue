<script>
import BaseComponent from 'primevue/basecomponent';
import { useStyle } from 'primevue/usestyle';

const styles = `
.p-inputtextarea-resizable {
    overflow: hidden;
    resize: none;
}

.p-fluid .p-inputtextarea {
    width: 100%;
}
`;

const classes = {
    root: ({ instance, props }) => [
        'p-inputtextarea p-inputtext p-component',
        {
            'p-filled': instance.filled,
            'p-inputtextarea-resizable ': props.autoResize
        }
    ]
};

const { load: loadStyle, unload: unloadStyle } = useStyle(styles, { id: 'primevue_textarea_style', manual: true });

export default {
    name: 'BaseTextarea',
    extends: BaseComponent,
    props: {
        modelValue: null,
        autoResize: Boolean
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
