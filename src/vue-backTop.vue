<template>
    <div :class="classes" :style="styles" @click="back">
        <slot>
            <div :class="innerClasses">
                <i class="el-icon-arrow-up"></i>
            </div>
        </slot>
    </div>
</template>
<script>
/*
* @params
* height        页面滚动高度达到该值时才显示BackTop组件
* bottom        组件距离底部的距离
* right         组件距离右部的距离
* duration      滚动动画持续时间，单位：毫秒
*
* @callback
* onClick       触发点击的回调
* complete      滚动到达顶部的回调
*/
import { requestAnimationFrame } from './RAFrame.js'
const prefixCls = 'vue-backTop'
export default {
    props: {
        height: {
            type: Number,
            default: 400
        },
        bottom: {
            type: Number,
            default: 30
        },
        right: {
            type: Number,
            default: 30
        },
        duration: {
            type: Number,
            default: 1000
        }
    },
    data() {
        return {
            visible: false
        }
    },
    computed: {
        classes() {
            return [`${prefixCls}`, { [`${prefixCls}-show`]: this.visible }]
        },
        styles() {
            return {
                bottom: `${this.bottom}px`,
                right: `${this.right}px`
            }
        },
        innerClasses() {
            return `${prefixCls}-inner`
        }
    },
    methods: {
        // 滚动方法
        scrollTop(from = 0, to = 0, duration = 500) {
            // 距离
            const difference = Math.abs(from - to)
            // 速度
            const step = Math.ceil(difference / duration * 50)

            const scroll = (start, end, step) => {
                if (start === end) {
                    this.$emit('complete')
                    return
                }
                // 每经过requestAnimationFrame执行一次（16.666ms）剩下的距离
                let d = (start - step < end) ? end : start - step
                window.scrollTo(d, d)
                requestAnimationFrame(() => scroll(d, end, step))
            }
            scroll(from, to, step)
        },
        // 滚动回调方法
        handleScroll() {
            // 滚动的距离大于设置的高度时，则显示backTop组件
            this.visible = window.pageYOffset >= this.height
        },
        // 点击backTop组件触发的方法
        back() {
            const sTop = window.pageYOffset
            this.scrollTop(sTop, 0, this.duration)
            this.$emit('onClick')
        }
    },
    mounted() {
        window.addEventListener('scroll', this.handleScroll, false)
        window.addEventListener('resize', this.handleScroll, false)
    },
    beforeDestroy() {
        window.removeEventListener('scroll', this.handleScroll, false)
        window.removeEventListener('resize', this.handleScroll, false)
    }
}
</script>
<style lang='stylus'>
$defaultColor = #42B983;
$defaultColorRGB = 66, 185, 131;
.vue-backTop {
    position: fixed;
    z-index: 800;
    cursor: pointer;
    display: none;
    &-show {
        display: block;
    }
    &-inner {
        border-radius: 2px;
        box-shadow: 0 1px 3px rgba($defaultColorRGB, 0.2);
        background-color: rgba($defaultColorRGB, 0.9);
        transition: all 0.2s ease-in-out;
        i {
            color: #fff;
            font-size: 24px;
            padding: 8px 12px;
            font-weight: bold;
        }
    }
}
</style>
