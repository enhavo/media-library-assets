<template>
    <div class="app-view">
        <flash-messages></flash-messages>
        <action-bar></action-bar>
        <input v-show="false" v-once :ref="(el) => setElement(el, 'upload')" multiple type="file">
        <div class="media-library-overlay" data-media-library-overlay>
            <div class="inner-content" data-scroll-container>
                <div class="media-library-progress" style="width: 0%;" v-bind:style="{'width': styleProgress()}"></div>
                <div class="tag-list">
                    <div class="headline"><i class="icon icon-filter_list"></i>
                        {{ translator.trans('enhavo_media_library.tags') }}
                    </div>
                    <tag-list
                        :tags="mediaLibrary.data.tags"
                    ></tag-list>

                    <div class="headline"><i class="icon icon-filter_list"></i>
                        {{ translator.trans('enhavo_media_library.content_types') }}
                    </div>
                    <content-type-list
                        v-if="mediaLibrary.data.contentTypes.length > 0"
                        :content-types="mediaLibrary.data.contentTypes"
                    ></content-type-list>
                </div>
                <div class="result-search">
                    <div class="search">
                        <input :ref="(el) => setElement(el, 'searchInput')" v-model="mediaLibrary.data.searchString" type="text" data-media-library-search />
                        <span class="media-library-search-reset" @click="clearSearch()"><i class="icon icon-close"></i></span>
                        <span class="media-library-search-submit" @click="search()"><i class="icon icon-search"></i></span>
                    </div>
                    <div v-if="!mediaLibrary.data.loading">
                        <div class="media-library-view-switch">
                            <i class="icon icon-account_box" :class="mediaLibrary.getView() == 'thumbnail' && 'active'" @click="setView('thumbnail')"></i>
                            <i class="icon icon-format_list_bulleted" :class="mediaLibrary.getView() == 'list' && 'active'" @click="setView('list')"></i>
                        </div>
                        <list-view v-if="mediaLibrary.data.view==='list'"></list-view>
                        <thumbnail-view v-else></thumbnail-view>
                        <div v-if="mediaLibrary.data.dropZoneActive" class="media-library-overlay">
                            Drop something here
                        </div>
                    </div>
                    <div v-else class="lds-ellipsis">
                        <div></div>
                        <div></div>
                        <div></div>
                        <div></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import {inject, onMounted, ref, watch} from "vue";
import Translator from "@enhavo/core/Translator";
import Router from "@enhavo/core/Router";
import MediaLibrary from "@enhavo/media-library/MediaLibrary";
import $ from 'jquery';
import '@enhavo/app/assets/styles/view.scss';
import "blueimp-file-upload/js/vendor/jquery.ui.widget.js";
import "blueimp-file-upload/js/jquery.fileupload";
import '@enhavo/media-library/assets/styles.scss';

const translator = inject<Translator>('translator');
const mediaLibrary = inject<MediaLibrary>('mediaLibrary');
const router = inject<Router>('router');

const searchString = ref<string>(mediaLibrary.data.searchString);

watch(searchString, (val) => {
    mediaLibrary.data.searchString = val;
})

const elements = {
    searchInput: null,
    upload:  null,
    itemList:  null,
    mediaLibrary: null,
}

onMounted(() => {
    window.addEventListener('keydown', (e) => {
        if (e.key == 'Enter') {
            let isSearchFocus = $(elements.searchInput).is(':focus');
            if (isSearchFocus) {
                search();
            }
        }
    });

    $(document).on('upload', function () {
        $(elements.upload).trigger('click');
    });

    $(elements.upload).fileupload({
        replaceFileInput: false,
        dataType: 'json',
        paramName: 'files',
        done: (event, data) => {
            if (false === data.response().result.success) {
                data.response().result.errors.forEach((error) => {
                    mediaLibrary.showError(error);
                });

                mediaLibrary.loaded();

            } else if (data.response().result.length === 0) {
                mediaLibrary.showError(translator.trans('enhavo_media_library.upload.fail.message'));
                mediaLibrary.loaded();

            } else {
                mediaLibrary.showSuccess(translator.trans('enhavo_media_library.upload.success.message'));
                mediaLibrary.refresh();
            }

            mediaLibrary.setProgress(0);
        },
        fail: (event, data) => {
            mediaLibrary.showError(translator.trans('enhavo_media_library.upload.fail.message'));
            mediaLibrary.setProgress(0);
            mediaLibrary.loaded();
        },
        add: (event, data) => {
            data.url = getRouter().generate(mediaLibrary.data.uploadRoute, {});
            data.submit();
            mediaLibrary.loading();
            mediaLibrary.setProgress(0);
        },
        progressall: (event, data) => {
            let progress = data.loaded / data.total * 100;
            mediaLibrary.setProgress(progress);
        },
        dropZone: elements.itemList,
        pasteZone: null
    });

    $(document).on('dragover', (e) => {
        e.preventDefault();
        e.stopPropagation();
        mediaLibrary.showDropZone();
    });

    $(elements.mediaLibrary).on('dragover', (e) => {
        e.preventDefault();
        e.stopPropagation();
        mediaLibrary.showDropZone();
        mediaLibrary.showDropZoneActive();
    });

    $(document).on('dragleave', (e) => {
        if ($(document).find('.app-view').length > 0 && $(document).find('.app-view').find(e.target).length > 0) return;
        e.preventDefault();
        e.stopPropagation();
        mediaLibrary.hideDropZone();
    });

    $(elements.mediaLibrary).on('dragleave', (e) => {
        e.preventDefault();
        e.stopPropagation();
        mediaLibrary.hideDropZoneActive();
    });

    $(document).on('drop', (e) => {
        e.preventDefault();
        e.stopPropagation();
        mediaLibrary.hideDropZone();
        mediaLibrary.hideDropZoneActive();
    });
});

function styleProgress() 
{
    return mediaLibrary.data.progress + '%';
}

function getAddLabel()
{
    if (mediaLibrary.data.multiple) {
        return "media_library.add_selecteded";
    } else {
        return "media_library.add_selected";
    }
}

function setView(type: string) 
{
    mediaLibrary.setView(type);
}

function search() 
{
    mediaLibrary.search();
}

function clearSearch() 
{
    mediaLibrary.clearSearch()

}

function getRouter() 
{
    return router;
}

function setElement(el: HTMLElement, property: string)
{
    elements[property] = el;
}
</script>
