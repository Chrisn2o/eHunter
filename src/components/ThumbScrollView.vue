<template>
    <div class="thumb-content">
        <awesome-scroll-view ref="scrollView" :is-hidden="true" class="thumb-scroll-view" @mouseenter="hover=true" @mouseleave="hover=false">
            <div class="header">
                <span class="app-name">E-HUNTER</span>
            </div>
            <!-- 160 is $thumb-view-height -->
            <div class="indicator" :style="{top: px(160*curIndex)}"></div>
            <div class="thumb-container" @click="select(index)" v-for="(item, index) of thumbs" ref="thumbContainers">
                <div class="thumb" :style="{background: `transparent url(${item.url}) -${item.offset}px 0 no-repeat`}"></div>
                <div class="hover-mask"></div>
                <div class="index">{{ index + 1 }}</div>
            </div>
        </awesome-scroll-view>
    </div>
</template>

<script>
    import { mapGetters, mapActions } from 'vuex'
    import ImgHtmlParser from 'src/service/parser/ImgHtmlParser.js'
    import AlbumCacheService from 'src/service/storage/AlbumCacheService.js'
    import AwesomeScrollView from './base/AwesomeScrollView.vue'

    export default {
        name: 'ThumbScrollView',

        data() {
            return {
                parser: new ImgHtmlParser(document.documentElement.innerHTML),
                info: window.info,
                imgList: [], // origin img list
                thumbs: [],
                curIndex: 0,
                hover: false
            }
        },

        components: {
            AwesomeScrollView
        },

        created() {
            // set Cookie for small thumb, important, but it is not work in content script. wired
            // this.initCookie()
                // .then(() => {
            this.initImgList();
                // });
        },

        computed: {
            ...mapGetters({
                centerIndex: 'curIndex'
            })
        },

        watch: {
            // centerIndex() {
            //     if (this.curIndex !== this.centerIndex && !this.hover) {
            //         this.curIndex = this.centerIndex;
            //         this.$refs.scrollView.ScrollTo(this.$refs.thumbContainers[this.centerIndex].offsetTop, 1000);
            //     }
            // },
            centerIndex: {
                handler: function(val, oldVal) {
                    if (this.curIndex !== this.centerIndex && !this.hover) {
                        this.curIndex = this.centerIndex;
                        this.$refs.scrollView.ScrollTo(this.$refs.thumbContainers[this.centerIndex].offsetTop, 1000);
                    }
                },
                deep: true
            }
        },

        methods: {
            ...mapActions([
                'setIndex'
            ]),

            select(index) {
                this.curIndex = index;
                this.setIndex(index);
            },

            initCookie() {
                return new Promise((resolve, reject) => {
                    chrome.cookies.set({
                        url: 'exhentai.org',
                        name: 'uconfig',
                        value: 'dm_t'
                    }, cookie => {
                        console.log(cookie);
                        resolve();
                    });
                });
            },

            deleteCookie(url, name, store, callback) {
                // console.log("Delete URL: "+url+" | NAME: "+name+" |");
                /* eslint-disable no-undef */
                chrome.cookies.remove({
                    'url': url,
                    'name': name,
                    'storeId': store
                }, function(details) {
                    if (typeof callback === 'undefined') {
                        return;
                    }
                    if (details === null || details === undefined || details === 'undefined') {
                        callback(false);
                    } else {
                        callback(true);
                    }
                })
            },

            initImgList() {
                AlbumCacheService.instance
                    .getThumbs(this.parser.getAlbumId(), this.parser.getIntroUrl(), this.parser.getSumOfPage())
                    .then(thumbs => {
                        console.log('thumbs');
                        console.log(thumbs);
                        this.thumbs = thumbs;
                    });
            }
        }
    }

</script>

<style lang="scss" scoped>
    @import "~style/_responsive";
    @import "~style/_variables";
    .thumb-content {
        position: relative;
        .thumb-scroll-view {
            position: relative;
            background: $thumb_scroll_view_bg;
            min-height: 100vh;
            height: 500px;
            display: inline-block;
            width: $thumb-view-width;
            >.header {
                position: relative;
                height: $header-height;
                background: $header-bg;
                >.app-name {
                    color: white;
                    font-weight: bolder;
                    font-size: 18px;
                    display: block;
                    position: absolute;
                    white-space: nowrap;
                    top: 50%;
                    left: 50%;
                    transform: translate(-50%, -50%);
                }
                .more-vertical-solid.icon {
                    display: block;
                    margin-top: 18px;
                    color: white;
                    position: absolute;
                    margin-left: 9px;
                    width: 2px;
                    height: 2px;
                    border-radius: 50%;
                    border: solid 1px currentColor;
                    background-color: currentColor;
                    &:before {
                        content: '';
                        position: absolute;
                        left: -1px;
                        top: -8px;
                        width: 2px;
                        height: 2px;
                        border-radius: 50%;
                        border: solid 1px currentColor;
                        background-color: currentColor;
                    }
                    &:after {
                        content: '';
                        position: absolute;
                        left: -1px;
                        top: 6px;
                        width: 2px;
                        height: 2px;
                        border-radius: 50%;
                        border: solid 1px currentColor;
                        background-color: currentColor;
                    }
                }
            }
            .thumb-container {
                position: relative;
                width: $thumb-view-width;
                padding: $thumb-view-margin 0;
                height: $thumb-view-height;
                text-align: center;
                display: flex;
                align-items: center;
                justify-content: center;
                box-sizing: border-box;
                >.thumb {
                    display: block;
                    width: $thumb-width;
                    // 1/1.44 is the default scale of ehentai's thumb
                    height: $thumb-width * 144 / 100;
                    transition: all 0.5s ease;
                }
                >.loc {
                    display: block;
                    color: rgba(white, .5);
                    font-size: 12px;
                }
                >.index {
                    display: none;
                }
                &:hover {
                    >.hover-mask {
                        position: absolute;
                        top: 0;
                        right: 0;
                        left: 0;
                        bottom: 0;
                        background: rgba($indicator_color, .2);
                    }
                    >.index {
                        position: absolute;
                        display: block;
                        font-weight: bolder;
                        font-size: 40px;
                        color: rgba($body_bg, .8);
                        top: 50%;
                        left: 50%;
                        transform: translate(-50%, -50%);
                        z-index: 20;
                        user-select: none;
                        cursor: default;
                    }
                }
            }
            .indicator {
                position: absolute;
                box-sizing: border-box;
                margin-top: $header-height;
                height: $thumb-view-height;
                left: 0;
                top: 0;
                right: 0;
                background: rgba($indicator_color, .3);
                border-left: 3px solid rgba($indicator_color, .5);
                border-right: 3px solid rgba($indicator_color, .5);
                transition: all 0.5s ease;
                z-index: 10;
                pointer-events: none;
            }
        }
    }

</style>
