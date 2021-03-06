<template>
    <DialogBase :dialog="dialog" :title="title" :fullscreen="$ui.mobile">
        <v-form
            ref="form"
            class="p-4"
            lazy-validation
            @submit.prevent="submit"
        >
            <slot :submit="submit" />
        </v-form>
        <template v-slot:actions>
            <slot name="secondary-actions" />
            <template v-if="!$ui.mobile">
                <v-spacer />
                <v-btn flat @click="close">Close</v-btn>
                <v-btn color="primary" type="submit" @click="submit">
                    {{ submitLabel }}
                </v-btn>
            </template>
        </template>
        <template v-slot:toolbar-actions>
            <v-btn
                v-if="$ui.mobile"
                dark
                flat
                small
                @click="submit"
            >
                {{ submitLabel }}
            </v-btn>
        </template>
    </DialogBase>
</template>

<script lang="ts">
import Vue from 'vue';

import { VForm } from 'vuetify';

import { Dialog } from '@/services/UI';

import DialogBase from '@/dialogs/DialogBase.vue';

export default Vue.extend({
    components: {
        DialogBase,
    },
    props: {
        title: {
            type: String,
            default: null,
        },
        dialog: {
            type: Object as () => Dialog,
            required: true,
        },
        cancelLabel: {
            type: String,
            default: 'Cancel',
        },
        submitLabel: {
            type: String,
            default: 'Submit',
        },
    },
    methods: {
        close() {
            this.$ui.closeDialog(this.dialog.id);
        },
        submit() {
            if ((this.$refs.form as VForm).validate()) {
                this.$emit('completed');
            }
        },
    },
});
</script>
