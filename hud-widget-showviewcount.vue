<!--
/*
**  HUDS ~~ Head-Up-Display Server (HUDS)
**  Copyright (c) 2020-2023 Dr. Ralf S. Engelschall <rse@engelschall.com>
**
**  Permission is hereby granted, free of charge, to any person obtaining
**  a copy of this software and associated documentation files (the
**  "Software"), to deal in the Software without restriction, including
**  without limitation the rights to use, copy, modify, merge, publish,
**  distribute, sublicense, and/or sell copies of the Software, and to
**  permit persons to whom the Software is furnished to do so, subject to
**  the following conditions:
**
**  The above copyright notice and this permission notice shall be included
**  in all copies or substantial portions of the Software.
**
**  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
**  EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
**  MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
**  IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
**  CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
**  TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
**  SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
*/
-->

<template>
    <div v-bind:style="style" class="showviewcount">
        <div ref="bar" class="bar">
            <div class="icon">
                <i class="fas fa-users"></i>
            </div>
            <div class="text">
                {{ viewcount }}
            </div>
        </div>
    </div>
</template>

<style lang="less" scoped>
.showviewcount {
    opacity: var(--opacity);
    .bar {
        height: 54px;
        margin: 20px;
        border-radius: 8px;
        padding: 4px;
        padding-left: 20px;
        background-color: var(--background);
        display: flex;
        flex-direction: row;
        .icon {
            padding-top: 4px;
            padding-right: 20px;
            color: var(--iconcolor);
            font-size: 30pt;
        }
        .text {
            padding-top: 6px;
            padding-left: 4px;
            font-family: "TypoPRO Fira Sans";
            font-weight: normal;
            font-size: 24pt;
            color: var(--textcolor);
        }
    }
}
</style>

<script>
export default {
    name: "showviewcount",
    props: {
        opacity:    { type: Number, default: 1.0 },
        background: { type: String, default: "" },
        iconcolor:  { type: String, default: "" },
        textcolor:  { type: String, default: "" }
    },
    data: () => ({
        viewcount: 0
    }),
    computed: {
        style: HUDS.vueprop2cssvar()
    },
    methods: {
        /*  receive the showviewcount events  */
        set (text) {
            this.viewcount = text
        },

        /*  allow the box to be animated  */
        animate () {
            const bar = this.$refs.bar
            /* soundfx.play("beep6") */
            const tl = anime.timeline({
                targets: bar,
                duration: 400,
                autoplay: true,
                direction: "normal",
                loop: 1,
                easing: "easeInOutSine"
            })
            tl.add({ scaleX: 1.10, scaleY: 1.20, translateY: -3, translateX: -2 })
                .add({ scaleX: 1.00, scaleY: 1.00, translateY: 0, translateX: 0 })
        }
    }
}
</script>

