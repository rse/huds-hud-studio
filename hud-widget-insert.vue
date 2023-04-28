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
    <div v-show="enabled" ref="insert" v-bind:style="style" class="insert">
        <div ref="bar" class="bar">
            <div class="head">
            </div>
            <div class="body">
                <span class="icon"><i v-bind:class="[ 'fas', 'fa-' + iconname ]"></i></span>
                {{ text }}
            </div>
            <div class="foot">
            </div>
        </div>
    </div>
</template>

<style lang="less" scoped>
.insert {
    opacity: var(--opacity);
    .bar {
        opacity: 0.0;
        font-family: "TypoPRO Fira Sans";
        font-weight: bold;
        font-size: 100pt;
        top: 550px;
        left: -260px;
        transform: rotate(-45deg);
        transform-origin: top left;
        position: absolute;
        .head {
            background-color: var(--background);
            height: 10px;
            margin-bottom: 10px;
        }
        .body {
            padding-top: 80px;
            background-color: var(--background);
            color: var(--titlecolor);
            text-align: center;
            height: 240px;
            width: 1200px;
            .icon {
                color: var(--iconcolor);
                padding-right: 10px;
            }
        }
        .foot {
            background-color: var(--background);
            height: 10px;
            margin-top: 10px;
        }
    }
}
</style>

<script>
export default {
    name: "insert",
    props: {
        opacity:    { type: Number, default: 1.0 },
        background: { type: String, default: "" },
        titletext:  { type: String, default: "" },
        titlecolor: { type: String, default: "" }
    },
    data: () => ({
        enabled:  false,
        progress: false,
        text : ""
    }),
    computed: {
        style: HUDS.vueprop2cssvar()
    },
    methods: {
        /*  set the text to be displayed in the insert */
        set (data) {
            this.text = data
        },

        /*  allow the box to be animated  */
        /***
        animate () {
            const bar = this.$refs.bar
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
        ***/

        /*  toggle insert on/off  */
        toggle () {
            /*  do nothing if we are still progressing  */
            if (this.progress)
                return
            this.progress = true

            /*  determine old/new state  */
            const el = this.$refs.bar
            const oldstate = this.enabled
            const newstate = !oldstate
            if (!oldstate)
                this.enabled = true

            /*  animate the insert  */
            const tl = anime.timeline({
                targets:  el,
                duration: 1000,
                autoplay: true
            })
            if (newstate) {
                /*  toggle insert on  */
                tl.add({
                    easing:     "easeOutBounce",
                    translateX: [ -400, 0.0 ],
                    rotate:     [ -45, -45 ],
                    opacity:    [ 1.0, 1.0 ]
                })
            }
            else {
                /*  toggle insert off  */
                tl.add({
                    easing:     "easeOutSine",
                    opacity:    [ 1.0, 0.0 ]
                })
            }
            tl.finished.then(() => {
                this.enabled  = newstate
                this.progress = false
            })
        }
    }
}
</script>
