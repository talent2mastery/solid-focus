<template>
    <v-list-tile
        :class="{
            'pending': !task.completed,
            'completed': task.completed,
            'bg-grey-lighter': $tasks.active === task,
            'cursor-pointer': !task.saving,
        }"
        class="task-item"
    >
        <v-list-tile-action class="flex-no-shrink">
            <v-checkbox
                v-if="!task.saving"
                :input-value="task.completed"
                color="primary"
                @change="toggleCompleted"
            />
            <v-progress-circular
                v-else
                :color="$ui.styles.colors.jade"
                :indeterminate="true"
                :size="24"
                :width="3"
                style="margin-right: 8px"
            />
        </v-list-tile-action>

        <v-list-tile-content
            :class="{ 'opacity-50': task.saving }"
            class="flex flex-row align-center flex-grow"
            @click="focus"
        >
            <div class="flex">
                <span class="truncate" v-html="renderedName" />
                <v-icon
                    v-if="task.description"
                    small
                    color="grey"
                    class="ml-1 opacity-75"
                >
                    description
                </v-icon>
            </div>
        </v-list-tile-content>

        <v-list-tile-action
            v-if="task.dueAt"
            :class="{
                'text-red': !task.completed && $dayjs(task.dueAt).isBefore($dayjs(), 'day'),
                'text-blue': !task.completed && $dayjs(task.dueAt).isSame($dayjs(), 'day'),
            }"
            class="flex flex-row flex-no-shrink flex-no-grow align-center ml-2 text-base"
            @click="focus"
        >
            {{ renderedDueAt }}
        </v-list-tile-action>

        <v-list-tile-action class="ml-2 flex-no-shrink">
            <v-btn
                :class="{ 'opacity-50': !task.starred }"
                :color="task.starred ? 'primary' : 'grey'"
                icon
                flat
                @click="toggleStarred"
            >
                <v-icon v-if="task.starred">star</v-icon>
                <v-icon v-else>star_outline</v-icon>
            </v-btn>
        </v-list-tile-action>
    </v-list-tile>
</template>

<script lang="ts">
import Vue from 'vue';

import Task from '@/models/soukai/Task';

import AsyncOperation from '@/utils/AsyncOperation';

export default Vue.extend({
    props: {
        task: {
            type: Object as () => Task,
            required: true,
        },
    },
    computed: {
        renderedName(): string {
            const html = this.$options.filters!.markdown(this.task.name);

            // Strip surrounding p tag
            return html.substring(3, html.length - 5);
        },
        renderedDueAt(): string {
            return this.$dayjs(this.task.dueAt).calendar(undefined, {
                sameDay: '[Today]',
                nextDay: '[Tomorrow]',
                nextWeek: 'dddd',
                lastDay: '[Yesterday]',
                lastWeek: '[Last] dddd',
                sameElse: 'DD/MM/YYYY',
            });
        },
    },
    methods: {
        focus({ target }: Event) {
            if (this.task.saving)
                return;

            if (target instanceof HTMLAnchorElement && target.getAttribute('target') === '_blank')
                return;

            this.$tasks.setActive(this.task);
        },
        async toggleCompleted() {
            // Allow the checkbox to be displayed as checked before the animation starts
            await this.$nextTick();

            await this.$ui.updateModel(
                this.task,
                task => task.toggleCompleted(),
                ['priority'],
            );
        },
        async toggleStarred() {
            await this.$ui.updateModel(
                this.task,
                task => task.toggleStarred(),
                ['priority'],
            );
        },
    },
});
</script>

<style lang="scss">
    // Minimum clickable area size in mobile devices
    $mobile-min-size: 32px;

    .task-item {
        height: 48px; // This is necessary for transitions to work correctly

        .v-list__tile__action {
            min-width: 0;
        }

        .v-list__tile__content {
            margin-left: config('margin.2');
        }

        &.completed .v-list__tile__content {
            opacity: config('opacity.75');
        }

        .layout-mobile & .v-list__tile__action i {
            font-size: $mobile-min-size;
            margin-left: calc((24px - #{$mobile-min-size}) / 2);
        }

    }
</style>
