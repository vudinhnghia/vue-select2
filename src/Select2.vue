<template>
    <div>
        <select class="form-control" :id="id" :name="name" :disabled="disabled" :required="required"></select>
    </div>
</template>

<script>
import $ from 'jquery';
import 'select2/dist/js/select2.full';
import 'select2/dist/css/select2.min.css'

export default {
    name: 'Select2',
    data() {
        return {
            select2: null,
            existEmitSelected: false
        };
    },
    emits: [
        'update:modelValue',
        'select'
    ],
    props: {
        modelValue: [String, Array], // previously was `value: String`
        id: {
            type: String,
            default: ''
        },
        name: {
            type: String,
            default: ''
        },
        placeholder: {
            type: String,
            default: ''
        },
        options: {
            type: Array,
            default: () => []
        },
        disabled: {
            type: Boolean,
            default: false
        },
        required: {
            type: Boolean,
            default: false
        },
        settings: {
            type: Object,
            default: () => {
            }
        },
    },
    watch: {
        options: {
            handler(val) {
                this.setOption(val);
            },
            deep: true
        },
        modelValue: {
            handler(val) {
                this.setValue(val);
            },
            deep: true
        },
    },
    methods: {
        init(objConfig = null) {
            if (this.select2) {
                $(this.$el).find('select').select2('destroy');
                $(this.$el).find('select').off('select2:select');
                $(this.$el).find('select').off('select2:unselect');
                $(this.$el).find('select').off('change');
                $(this.$el).find('select').off('select2:clear');
                this.select2.empty();
            }

            let settings = {
                placeholder: this.placeholder,
                ...this.settings,
                data: this.options
            }

            if(objConfig){
                settings = {
                    ...settings,
                    ...objConfig
                }
            }

            this.select2 = $(this.$el)
                .find('select')
                .select2(settings)
                .on('select2:select', ev => {
                    this.$emit('update:modelValue', this.select2.val());
                    this.$emit('select', ev['params']['data']);
                })
                .on('select2:unselect', ev => {
                    this.$emit('update:modelValue', this.select2.val());
                    this.$emit('unselect', ev['params']['data']);
                })
                .on('change', ev => {
                    this.$emit('change', ev);
                })
                .on('select2:clear', ev => {
                    this.$emit('clear');
                });
            this.setValue(this.modelValue);
        },
        setOption(val = []) {
            this.select2.empty();
            this.select2.select2({
                placeholder: this.placeholder,
                ...this.settings,
                data: val
            });
            this.setValue(this.modelValue);
        },
        setValue(val) {
            if (val instanceof Array) {
                this.select2.val([...val]);
            } else {
                this.select2.val([val]);
            }
            this.select2.trigger('change');
        }
    },
    mounted() {
        this.init();
    },
    beforeUnmount() {
        this.select2.select2('destroy');
    }
};
</script>
