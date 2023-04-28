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
    <div>
        <input type="text" v-model="text"/>
        <button v-on:click="sendText">Send message</button>
    </div>
</template>

<style lang="less" scoped>
.insert {
}
</style>

<script>
export default {
    name: "title-bar",
    props: {
        questionbackground:    { type: String, default: "" },
        questiontitlecolor:    { type: String, default: "" },
        questionmessagecolor:  { type: String, default: "" },
        objectionbackground:   { type: String, default: "" },
        objectiontitlecolor:   { type: String, default: "" },
        objectionmessagecolor: { type: String, default: "" },
        commentbackground:     { type: String, default: "" },
        commenttitlecolor:     { type: String, default: "" },
        commentmessagecolor:   { type: String, default: "" },
        privacylevel:          { type: String, default: "" }
    },
    data: () => ({
        text:      "",
        queue:     [],
        popups:    [],
        attendees: {},
        timer:     null
    }),
    computed: {
        style: HUDS.vueprop2cssvar()
    },
    setup () {
        const boxes = Vue.ref([])
        Vue.onBeforeUpdate(() => { boxes.value = [] })
        return { boxes }
    },
    methods: {
        /*  add a insert */
        async sendText (text) {
            console.log(this)
            console.log(">>>sendText " + text)
            const data = { text: this.text }
            console.log(">>>execute set")
            huds.send("insert",data)
        },
    },
    created () {
        /*  track the attendees (similar to "attendance" widget to be in sync)  */
        this.timer = setInterval(() => {
            /*  expire attendees not seen recently
                (refresh usually every 10min, but we accept also up to 20min)  */
            const now = (new Date()).getTime()
            for (const client of Object.keys(this.attendees)) {
                const seen = this.attendees[client].seen
                if (seen + ((20 + 2) * 60 * 1000) < now)
                    delete this.attendees[client]
            }
        }, 2 * 1000)

        /*  queue worker loop  */
        const progress = async () => {
            while (this.queue.length > 0) {
                const cmd = this.queue.shift()
                try {
                    await this[cmd.method](...cmd.args)
                }
                catch (err) {
                    /*  no-op  */
                }
            }
            this.timer = setTimeout(progress, 50)
        }
        this.timer = setTimeout(progress, 50)
    }
}
</script>

