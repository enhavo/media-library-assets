<template>
    <div class="media-library-sort">
        <ul>
            <li v-for="column in mediaLibrary.data.columns"
                 @click="onSort(column)"
                 :class="isSorted(column)?'sorted':''"
                 >
                {{ column.label }}
                <i aria-hidden="true"
                   class="icon sortable"
                   :class="sortClass(column)">
                </i>
            </li>
        </ul>
    </div>
    <pagination v-if="mediaLibrary.hasPagination()"></pagination>
    <ul class="images scroll-container">
        <file-thumbnail
            v-for="file of mediaLibrary.data.files"
            :file="file"
        ></file-thumbnail>
    </ul>
    <pagination v-if="mediaLibrary.hasPagination()"></pagination>
</template>

<script setup lang="ts">
import {inject} from "vue";
import MediaLibrary from "@enhavo/media-library/MediaLibrary";
import {Column} from "@enhavo/media-library/Data";
import '@enhavo/app/assets/styles/view.scss';

const mediaLibrary = inject<MediaLibrary>('mediaLibrary');

function onSort(column: Column)
{
    mediaLibrary.setSortColumn(column);
}

function isSorted(column: Column)
{
    return mediaLibrary.isSortedColumn(column);
}

function sortClass(column: Column)
{
    if (mediaLibrary.isSortedColumn(column)) {
        return mediaLibrary.data.sortColumn.direction === 'asc' ? 'icon-arrow_upward' : 'icon-arrow_downward';
    }
}
</script>
