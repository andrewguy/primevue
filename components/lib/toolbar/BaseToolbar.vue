<script>
import BaseComponent from 'primevue/basecomponent';
import { useStyle } from 'primevue/usestyle';

const styles = `
.p-toolbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
}

.p-toolbar-group-start,
.p-toolbar-group-center,
.p-toolbar-group-end {
    display: flex;
    align-items: center;
}

.p-toolbar-group-left,
.p-toolbar-group-right {
    display: flex;
    align-items: center;
}
`;

const classes = {
    root: 'p-toolbar p-component',
    start: 'p-toolbar-group-start p-toolbar-group-left',
    center: 'p-toolbar-group-center',
    end: 'p-toolbar-group-end p-toolbar-group-right'
};

const { load: loadStyle, unload: unloadStyle } = useStyle(styles, { id: 'primevue_toolbar_style', manual: true });

export default {
    name: 'BaseToolbar',
    extends: BaseComponent,
    props: {
        'aria-labelledby': {
            type: String,
            default: null
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
