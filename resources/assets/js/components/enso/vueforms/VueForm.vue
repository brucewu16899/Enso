<template>

    <div>
        <h5 class="title is-5"
            v-if="data.icon || data.title">
            <span class="icon"
                v-if="data.icon">
                <fa :icon="data.icon"></fa>
            </span>
            <span v-if="data.title">
                {{ data.title }}
            </span>
        </h5>
        <form class="is-marginless"
            @submit.prevent="submit()">
            <div class="columns is-multiline"
                v-for="(section, index) in data.sections"
                :key="index">
                <div class="column is-12"
                    v-if="section.divider">
                    <divider :title="i18n(section.title)"
                        :placement="data.dividerTitlePlacement">
                    </divider>
                </div>
                <div v-for="field in section.fields"
                    :class="[ 'column',
                        section.columns !== 'custom'
                            ? columnSize(section.columns)
                            : `is-${field.column}`
                    ]"
                    :key="field.name"
                    v-if="!field.meta.hidden">
                    <div class="field">
                        <label class="label">
                            {{ i18n(field.label) }}
                            <span class="icon is-small has-text-info"
                                v-if="field.meta.tooltip"
                                v-tooltip="field.meta.tooltip">
                                <fa icon="info" size="xs"></fa>
                            </span>
                            <p v-if="errors.has(field.name)"
                                class="help is-danger is-pulled-right">
                                {{ errors.get(field.name) }}
                            </p>
                        </label>
                        <span v-if="field.meta.custom">
                            <slot :name="field.name"
                                :field="field"
                                :errors="errors">
                            </slot>
                        </span>
                        <span v-else>
                            <div :class="['control', { 'has-icons-right': errors.has(field.name) }]"
                                v-if="
                                    field.meta.type === 'input'
                                    && field.meta.content !== 'checkbox'
                                ">
                                <input
                                    class="input"
                                    v-model="field.value"
                                    :type="field.meta.content"
                                    :class="{ 'is-danger': errors.has(field.name) }"
                                    :readonly="field.meta.readonly"
                                    :disabled="field.meta.disabled"
                                    @keydown="$emit('update');"
                                    @input="errors.clear(field.name);"
                                    :step="field.meta.step"
                                    :min="field.meta.min"
                                    :max="field.meta.max">
                                <span class="icon is-small is-right has-text-danger"
                                    v-if="errors.has(field.name)">
                                    <fa icon="exclamation-triangle"></fa>
                                </span>
                            </div>
                            <vue-switch v-model="field.value"
                                size="is-large"
                                type="is-success"
                                :disabled="field.meta.disabled || field.meta.readonly"
                                @click="$emit('update')"
                                v-if="
                                    field.meta.type === 'input'
                                    && field.meta.content === 'checkbox'
                                ">
                            </vue-switch>
                            <vue-select v-if="field.meta.type === 'select'"
                                v-model="field.value"
                                @input="errors.clear(field.name);"
                                :i18n="i18n"
                                :has-error="errors.has(field.name)"
                                :label="field.meta.label || 'name'"
                                :track-by="field.meta.trackBy || 'id'"
                                :options="field.meta.options"
                                :source="field.meta.source"
                                :multiple="field.meta.multiple"
                                :disabled="field.meta.disabled"
                                :placeholder="field.meta.placeholder">
                            </vue-select>
                            <datepicker v-if="field.meta.type === 'datepicker'"
                                @input="errors.clear(field.name)"
                                v-model="field.value"
                                :format="field.meta.format"
                                :time="field.meta.time"
                                :disabled="field.meta.disabled">
                            </datepicker>
                            <datepicker v-if="field.meta.type === 'timepicker'"
                                @input="errors.clear(field.name)"
                                v-model="field.value"
                                :format="field.meta.format"
                                time-only
                                :disabled="field.meta.disabled">
                            </datepicker>
                            <div class="control has-icons-right"
                                v-if="field.meta.type === 'textarea'">
                                <textarea @input="errors.clear(field.name)"
                                    class="textarea"
                                    :class="{ 'is-danger': errors.has(field.name) }"
                                    v-model="field.value"
                                    :rows="field.meta.rows"
                                    :disabled="field.meta.disabled">
                                </textarea>
                                <span class="icon is-small is-right has-text-danger"
                                    v-if="errors.has(field.name)">
                                    <fa icon="exclamation-triangle"></fa>
                                </span>
                            </div>
                        </span>
                    </div>
                </div>
            </div>
            <button class="button"
                v-if="data.actions.destroy"
                :disabled="data.actions.destroy.forbidden"
                :class="data.actions.destroy.button.class"
                @click.prevent="modal = true">
                <span>{{ i18n(data.actions.destroy.button.label) }}</span>
                <span class="icon">
                    <fa :icon="data.actions.destroy.button.icon"></fa>
                </span>
            </button>
            <button class="button"
                :class="data.actions.create.button.class"
                @click.prevent="create()"
                v-if="data.actions.create"
                :disabled="data.actions.create.forbidden">
                <span>{{ i18n(data.actions.create.button.label) }}</span>
                <span class="icon">
                    <fa :icon="data.actions.create.button.icon"></fa>
                </span>
            </button>
            <button type="submit"
                v-if="data.actions.store"
                class="button is-pulled-right"
                :class="[data.actions.store.button.class, { 'is-loading': loading }]"
                :disabled="data.actions.store.forbidden || errors.any()">
                <span>{{ i18n(data.actions.store.button.label) }}</span>
                <span class="icon">
                    <fa :icon="data.actions.store.button.icon"></fa>
                </span>
            </button>
            <button type="submit"
                v-if="data.actions.update"
                class="button is-pulled-right"
                :class="[data.actions.update.button.class, { 'is-loading': loading }]"
                :disabled="data.actions.update.forbidden || errors.any()">
                <span>{{ i18n(data.actions.update.button.label) }}</span>
                <span class="icon">
                    <fa :icon="data.actions.update.button.icon"></fa>
                </span>
            </button>
            <div class="is-clearfix"></div>
        </form>
        <modal v-if="data.actions.destroy"
            :show="modal"
            :i18n="i18n"
            :message="data.actions.destroy.button.message"
            @close="modal = false"
            @commit="destroy()">
        </modal>
    </div>

</template>

<script>

import { VTooltip } from 'v-tooltip';
import fontawesome from '@fortawesome/fontawesome';
import { faTrashAlt, faPlus, faCheck, faExclamationTriangle, faUndo, faInfo }
    from '@fortawesome/fontawesome-free-solid/shakable.es';
import Divider from './Divider.vue';
import Errors from './classes/Errors';
import Modal from './Modal.vue';
import VueSwitch from './VueSwitch.vue';
import VueSelect from '../select/VueSelect.vue';
import Datepicker from './Datepicker.vue';

fontawesome.library.add(faTrashAlt, faPlus, faCheck, faExclamationTriangle, faUndo, faInfo);

export default {
    name: 'VueForm',

    directives: { tooltip: VTooltip },

    components: {
        Divider, VueSwitch, Modal, VueSelect, Datepicker,
    },

    props: {
        data: {
            type: Object,
            required: true,
        },
        params: {
            type: Object,
            default: null,
        },
        i18n: {
            type: Function,
            default(key) {
                return typeof this.__ === 'function'
                    ? this.__(key)
                    : key;
            },
        },
    },

    data() {
        return {
            loading: false,
            modal: false,
            errors: new Errors(),
        };
    },

    computed: {
        path() {
            return this.data.method === 'post'
                ? this.data.actions.store.path
                : this.data.actions.update.path;
        },
    },

    methods: {
        columnSize(columns) {
            return `is-${parseInt(12 / columns, 10)}`;
        },
        create() {
            this.$emit('create');
            this.$router.push({ name: this.data.actions.create.route });
        },
        submit() {
            this.loading = true;

            axios[this.data.method](this.path, this.formData()).then(({ data }) => {
                this.loading = false;
                this.$toastr.success(data.message);
                this.$emit('submit');

                if (data.redirect) {
                    this.$router.push({
                        name: data.redirect,
                        params: { id: data.id },
                    });
                }
            }).catch((error) => {
                const { status, data } = error.response;
                this.loading = false;

                if (status === 422) {
                    this.errors.set(data.errors);

                    return;
                }

                this.handleError(error);
            });
        },
        formData() {
            return this.data.sections
                .reduce((fields, section) => fields
                    .concat(section.fields), [])
                .reduce((object, field) => {
                    object[field.name] = field.value;
                    return object;
                }, { _params: this.params });
        },
        destroy() {
            this.modal = false;
            this.loading = true;

            axios.delete(this.data.actions.destroy.path).then(({ data }) => {
                this.loading = false;
                this.$toastr.success(data.message);
                this.$emit('destroy');

                if (data.redirect) {
                    this.$router.push({ name: data.redirect });
                }
            }).catch((error) => {
                this.loading = false;
                this.handleError(error);
            });
        },
        field(field) {
            return this.data.sections
                .reduce((fields, section) => fields.concat(section.fields), [])
                .find(item => item.name === field);
        },
    },
};

</script>

<style lang="scss" scoped>

    .title {
        .icon {
            vertical-align: text-bottom;
        }
    }

</style>
