<template>
    <div class="view-table-row" :class="{'active': file.selected}"
        @click="onClick()"
    >
        <div class="view-table-row-columns">
            <div class="view-table-col-text view-table-col" style="width: 25%">
                {{ file.label }}
            </div>
            <div class="view-table-col-text view-table-col" style="width: 25%">
                {{ file.suffix }}
            </div>
            <div class="view-table-col-text view-table-col" style="width: 25%">
                {{ file.type }}
            </div>
            <div class="view-table-col-text view-table-col" style="width: 25%">
                {{ file.date }}
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import {inject} from "vue";
import {File} from "@enhavo/media-library/Data";
import MediaLibrary from "@enhavo/media-library/MediaLibrary";

const mediaLibrary = inject<MediaLibrary>('mediaLibrary');

const props = defineProps<{
    file: File
}>()

function onClick()
{
    mediaLibrary.open(props.file);
}
</script>
