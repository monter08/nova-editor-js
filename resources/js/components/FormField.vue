<template>
    <DefaultField
        :field="currentField"
        :errors="errors"
        :show-help-text="showHelpText"
        :full-width-content="true"
        @keydown.stop
    >
        <template #field>
            <div
                :id="`editor-js-${currentField.attribute}`"
                ref="input"
                class="editor-js"
            />
        </template>
    </DefaultField>
</template>

<script>
import { DependentFormField, HandlesValidationErrors } from "laravel-nova";

export default {
    mixins: [HandlesValidationErrors, DependentFormField],

    data() {
        return {
            editorInstance: null,
            isMounted: false,
        };
    },

    mounted() {
        this.isMounted = true;
        this.initEditor();
    },

    beforeUnmount() {
        if (this.editorInstance) {
            this.editorInstance.destroy();
            this.editorInstance = null;
        }
    },

    methods: {
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.value = this.currentField.value;

            if (this.isMounted) {
                this.initEditor();
            }
        },

        initEditor() {
            const self = this;

            this.$nextTick(() => {
                if (self.editorInstance) {
                    self.editorInstance.destroy();
                    self.editorInstance = null;
                }

                const currentContent =
                    typeof self.currentField.value === "object"
                        ? self.currentField.value
                        : JSON.parse(self.currentField.value);

                self.editorInstance = NovaEditorJS.getInstance(
                    {
                        /**
                         * Wrapper of Editor
                         */
                        holder: `editor-js-${self.currentField.attribute}`,

                        /**
                         * This Tool will be used as default
                         */
                        defaultBlock:
                            self.currentField.editorSettings.initialBlock,

                        /**
                         * Default placeholder
                         */
                        placeholder:
                            self.currentField.editorSettings.placeholder,

                        /**
                         * Enable autofocus
                         */
                        autofocus: self.currentField.editorSettings.autofocus,

                        /**
                         * Internalization config
                         */
                        i18n: {
                            /**
                             * Text direction. If not set, uses ltr
                             */
                            direction:
                                self.currentField.editorSettings.rtl ?? false
                                    ? "rtl"
                                    : "ltr",
                        },

                        /**
                         * Initial Editor data
                         */
                        data: currentContent,

                        /**
                         * Min height of editor
                         */
                        minHeight: 35,

                        onReady() {},
                        onChange() {
                            self.editorInstance.save().then((savedData) => {
                                self.handleChange(savedData);
                            });
                        },
                    },
                    self.field
                );
            });
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            const value =
                typeof this.value === "string"
                    ? this.value
                    : JSON.stringify(this.value);
            this.fillIfVisible(
                formData,
                this.currentField.attribute,
                value || ""
            );
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = JSON.stringify(value);
        },
    },
};
</script>
