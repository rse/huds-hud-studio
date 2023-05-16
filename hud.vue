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
    <div class="hud" v-bind:class="{ minimize: minimize }">
        <background v-if="debug" ref="background" class="background"
        ></background>
        <insert ref="insert" class="insert"
            v-bind:opacity="config.insert.opacity"
            v-bind:background="config.insert.background"
            v-bind:titletext="insert.titletext"
            v-bind:titlecolor="insert.titlecolor"
        ></insert>
        <logo ref="logo" class="logo"
        ></logo>
    </div>
</template>

<style lang="less" scoped>
html,
body {
    background-color: transparent;
}
.hud {
    width: 100vw;
    height: 100vh;
    position: relative;
    font-family: sans-serif;
    font-size: 22pt;
    overflow: hidden;
    > .background {
        position: absolute;
        width: 100vw;
        height: 100vh;
    }
    &.minimize {
        > .insert {
            display: none;
        }
    }
}
</style>

<script>
import ctl from './ctl.vue'

export default {
    name: "hud",
    data: () => ({
        insert:   false,
        config:   huds.config(),
        debug:    typeof huds.options.debug === "boolean" ? huds.options.debug : false,
        banner:   null,
        logo:     false,
        minimize: false
    }),
    components: {
        "background":    Vue.loadComponent("hud-widget-background.vue"),
        "insert":        Vue.loadComponent("hud-widget-insert.vue"),
        "logo":          Vue.loadComponent("hud-widget-logo.vue"),
    },
    created () {
        /*  interaction for insert widget  */
        huds.bind("insert", (event, data) => {
            const a = this.$refs.insert
            if (!(   typeof data.text  === "string" && data.text !== ""))
                return
            a.set(data.text)
        })

        huds.bind("logo", (event, data) => {
            const a = this.$refs.logo
            a.toggle(data)
        })

        huds.bind("getStatusDisabled", (event) => {
            const a = this.$refs.insert
            a.getStatusDisabled()
        })
        /*  interaction for minimization  */
        Mousetrap.bind("m", (e) => {
            huds.send("minimize.toggle")
        })
        huds.bind("minimize.toggle", (event, data) => {
            this.minimize = !this.minimize
        })


        /*  control peers  */
        huds.bind("peer.reconnect", () => {
            huds.send("reconnect", {}, this.config.id.peer)
        })
        huds.bind("peer.disconnect", () => {
            huds.send("disconnect", {}, this.config.id.peer)
        })
    },
    methods: {
    }
}
</script>

