<script>
    const INIT = 0;
    const SCROLL = 1;
    const SWIPE = 2;
    export default {
        props: {
            indexTab: {
                type: String
            },
            swipeDistance: {
              type: Number,
              default: 100
            }
        },
        created() {
            this.tabs = this.$children;
        },
        mounted(){

            this.$nextTick(() => {
                this.children = document.querySelectorAll('.tabs-panel-content');
                if(this.tabs.length) {
                    // this.indexTab ? this.selectTab(this.indexTab) : this.selectTab(this.tabs[0].tabHash);
                    if(this.indexTab) {
                        // this.selectTab(this.indexTab);
                        this.tabs.forEach((item, idx) => {
                            item.tabHash === this.indexTab && this.setPage(idx);
                        });
                    } else {
                        this.selectTab(this.tabs[0].tabHash);
                        this.setPage(0);
                    }
                }
            });
            this.initDPR();
        },
        watch: {
            'indexTab'() {
                if(this.tabs.length) {
                    this.indexTab ? this.selectTab(this.indexTab) : this.selectTab(this.tabs[0].tabHash);
                }
            }
        },
        data() {
            return {
                tabs: [],
                children: [],
                activeHash: '',
                touchPoint: {
                    startLeft: 0,
                    startTop: 0,
                    startTime: 0
                },
                distance: {
                    left: 0,
                    top: 0
                },
                currentPage: 0,
                translateX: 0,
                transitionOrnot: false,
                startTranslateX: 0,
                swipeType: INIT,
                dpr: 1,
                swipeTopDistance: 60
            }
        },
        methods: {
            selectTab(hash){
                // console.log(hash);
                this.tabs.forEach((tab) => {
                    tab.isActive = (tab.tabHash === hash);
                });
                this.activeHash = hash;
            },
            next() {
                let currentpage = this.currentPage;
                (currentpage < this.tabs.length - 1) && currentpage++;
                // console.log('next, currentpage: ' + currentpage);
                this.setPage(currentpage);
            },
            prev() {
                let currentpage = this.currentPage;
                (currentpage > 0) && currentpage--;
                // console.log('prev, currentpage: ' + currentpage);
                this.setPage(currentpage);
            },
            reset() {
                this.setPage(this.currentPage);
            },
            setPage(page) {
                // console.log('set');
                const prevPage = this.currentPage;
                this.currentPage = page;

                this.translateX = -this.tabs.reduce(function(total, item, i){

                    return i > page - 1 ? total : total + item.$el['clientWidth'];
                }, 0);

                // console.log(this.currentPage);
                [].forEach.call(this.children, (el, index) => {

                    if(index === this.currentPage) {
                        el.style.height = 'auto';
                    } else {
                        el.style.height = '0px';
                    }

                });
                this.selectTab(this.tabs[page].tabHash);
                this.currentPage !== prevPage && this.$emit('changePage', this.currentPage);
                setTimeout(() => {
                    if(!this.transitionOrnot) {
                        this.transitionOrnot = true;
                    }
                });
            },
            onTouchStart(event) {
                const touchPoint = event.changedTouches[0] || event.touches[0];

                const startLeft = touchPoint.clientX;
                // console.log('left:' + startLeft);
                this.touchPoint.startLeft = startLeft;

                const startTop = touchPoint.clientY;
                // console.log('top:' + startTop);
                this.touchPoint.startTop = startTop;

                const startTranslateX = this.translateX;
                this.startTranslateX = startTranslateX;

                const touchTime = new Date().getTime();
                // console.log('time:' + touchTime);
                this.touchPoint.startTime = touchTime;
            },
            onTouchMove(event) {
                const touchPoint = event.changedTouches[0] || event.touches[0];
                const distanceLeft = touchPoint.clientX - this.touchPoint.startLeft;
                // console.log('distance:' + distanceLeft);
                this.distance.left = distanceLeft;
                const distanceTop = Math.abs(touchPoint.clientY - this.touchPoint.startTop);
                // console.log('distanceTop:' + distanceTop);
                this.distance.top = distanceTop;

                switch(this.swipeType) {
                    case INIT:
                        if(Math.abs(distanceLeft) / distanceTop > 1.5) {
                            event.preventDefault();
                            event.stopPropagation();
                            this.swipeType = SWIPE;
                        } else {
                            this.swipeType = SCROLL;
                        }
                        break;
                    case SCROLL:
                        break;
                    case SWIPE:
                        this.translateX = this.startTranslateX + distanceLeft;
                        break;
                }
            },
            onTouchEnd(event) {
                var quick = new Date().getTime() - this.startTime < 1000;
                // console.log(quick);
                // console.log(this.distance.top);
                if(Math.abs(this.distance.left) > Math.abs(this.distance.top) && (this.distance.left < -(this.swipeDistance * this.dpr) && this.distance.top < (this.swipeTopDistance * this.dpr)) && (this.distance.left < -15 && this.distance.top / this.distance.left > -5)) {
                    console.log('next');
                    // console.log(-(100 * this.dpr));
                    this.next();
                } else if(Math.abs(this.distance.left) > Math.abs(this.distance.top) && (this.distance.left > (this.swipeDistance * this.dpr) && this.distance.top < (this.swipeTopDistance * this.dpr)) && (this.distance.left > 15 && this.distance.top / this.distance.left < 5)) {
                    // console.log(this.distance.left);
                    // console.log(100 * this.dpr);
                    console.log('prev');
                    this.prev();
                } else {
                    this.reset();
                }
                this.swipeType = INIT;
                this.distance.left = 0;
                this.distance.top = 0;
            },
            initDPR() {
                var win = window;
                var isAndroid = win.navigator.appVersion.match(/android/gi);
                var isIPhone = win.navigator.appVersion.match(/iphone/gi);
                var devicePixelRatio = win.devicePixelRatio;
                if(isIPhone) {
                    if(devicePixelRatio >= 3 && this.dpr) {
                        this.dpr = 3;
                    } else if(devicePixelRatio >= 2 && this.dpr){
                        this.dpr = 2;
                    } else {
                        this.dpr = 1;
                        // this.dpr = 1.5;
                    }
                } else {
                    this.dpr = 1;
                    //this.dpr = 1.5;
                }
            }
        }
    }
</script>

<template>
    <div class="tabs-container">
        <ul class="tabs-list" role="tablist">
            <li v-for="(tab,index) in tabs"
                :key="index"
                class="tabs-title"
                :style="{width: (100 / tabs.length) + '%', fontSize: tab.fontSize, height: tab.tabHeight}">
                <a
                    :class="activeHash === tab.tabHash ? 'active' : ''"
                    :style="{ lineHeight: tab.tabHeight }"
                    class="tabs-titleLink"
                    @touchstart="setPage(index)"
                    @click="setPage(index)">
                    {{ tab.tabHeader }}
                    <i class="cursor" :style="{ 'background': tab.color ? tab.color : '#fdde02' }"></i>
                </a>
            </li>
        </ul>

        <div class="tabContent-wrap"
            @touchstart="onTouchStart($event)"
            @touchmove="onTouchMove($event)"
            @touchend="onTouchEnd($event)"
            >
            <div class="tabs-panel"
            :class="{ 'transition': transitionOrnot }"
            :style="{ 'transform': `translate3d(${translateX}px, 0, 0)` }"
            >
                <slot />
            </div>
        </div>

    </div>
</template>

<style lang="scss" scoped>
    $font_size : 75;

    @function rem($pixels){
        @return $pixels / $font_size + rem;
    }

    .tabs-container {
        width: 100%;
        .tabs-panel {
            width: 100%;
            height: 100%;
            font-size: 0;
            white-space: nowrap;
            // transition: all 0.2s ease;
            &.transition {
                transition: all 0.2s ease;
            }
            .tabs-panel-content {
                display: inline-block;
                width: 100%;
                font-size: rem(12);
                vertical-align: top;
                -webkit-overflow-scrolling: touch;
                transform: translate3d(0,0,0);
            }
        }
        .tabContent-wrap {
            width: 100%;
            overflow: hidden;
        }
        .tabs-list {
            position: relative;
            margin: 0;
            padding: 0;
            border-bottom: 1px solid #ccc;
            list-style: none;
            overflow: hidden;
        }
        .tabs-title {
            position: relative;
            float: left;
            width: 50%;
            height: rem(90);
            line-height: rem(90);
            text-align: center;
            font-size: rem(36);
            &:after {
                content: '';
                display: block;
                position: absolute;
                top: 50%;
                right: 0;
                transform: translateY(-50%);
                width: rem(3);
                height: rem(36);
                background: #e6e6e6;
            }
            &:last-child:after {
                display: none;
            }
            .tabs-titleLink {
                display: block;
                width: 100%;
                height: 100%;
                position: relative;
                color: #999;
                &:hover {
                    text-decoration: none;
                    color: #000;
                }
                &:focus {
                    outline: none;
                }
                &.active {
                    color: #000;
                    font-weight: 900;
                }
                .cursor {
                    content: '';
                    display: none;
                    position: absolute;
                    bottom: 0;
                    left: 50%;
                    transform: translateX(-50%);
                    width: 100%;
                    height: rem(8);
                    background: #fdde02;
                }
                &.active .cursor {
                    display: block;
                }
            }
        }
    }
</style>
