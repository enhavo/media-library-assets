<template>
    <li :class="{'active':contentType.selected}">
        <div class="tag-name" @click="onClick()">{{ contentType.label }}</div>
    </li>
</template>

<script setup lang="ts">
import {inject} from "vue";
import {ContentType} from "@enhavo/media-library/Data";
import MediaLibrary from "@enhavo/media-library/MediaLibrary";

const mediaLibrary = inject<MediaLibrary>('mediaLibrary');

const props = defineProps<{
    contentType: ContentType
}>()

function onClick()
{
    mediaLibrary.setActiveContentType(props.contentType);
}
</script>
