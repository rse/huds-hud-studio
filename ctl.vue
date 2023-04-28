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
    <div class="ctl">
        <popup ref="popup" class="popup"
            v-bind:questionbackground="config.popup.questionbackground"
            v-bind:questiontitlecolor="config.popup.questiontitlecolor"
            v-bind:questionmessagecolor="config.popup.questionmessagecolor"
            v-bind:objectionbackground="config.popup.objectionbackground"
            v-bind:objectiontitlecolor="config.popup.objectiontitlecolor"
            v-bind:objectionmessagecolor="config.popup.objectionmessagecolor"
            v-bind:commentbackground="config.popup.commentbackground"
            v-bind:commenttitlecolor="config.popup.commenttitlecolor"
            v-bind:commentmessagecolor="config.popup.commentmessagecolor"
            v-bind:privacylevel="config.popup.privacylevel"
        ></popup>
    </div>
</template>

<style lang="less" scoped>
.ctl {
    width: 100vw;
    height: 100vh;
    position: relative;
    font-family: sans-serif;
    font-size: 12pt;
    overflow: hidden;
    background-color: #333333;
    color: #ffffff;
    > .attendance {
        position: absolute;
        right: 520px;
        bottom: 20px;
        width: 270px;
    }
    > .attendees {
        position: absolute;
        top: 20px;
        left: 20px;
        width:  calc(100vw - 830px);
        height: calc(100vh - 40px);
    }
    > .feeling {
        position: absolute;
        right: 520px;
        bottom: 100px;
        width: 270px;
    }
    > .timer {
        position: absolute;
        right: 20px;
        bottom: 20px;
        width: 470px;
        height: 470px;
    }
    > .agenda {
        position: absolute;
        top: 20px;
        right: 30px;
        width: 760px;
        height: calc(100vh - 530px);
    }
    > .popup {
        position: absolute;
        bottom: 40px;
        width: calc(50vw);
        height: calc(100vh - 80px);
        left: 40px;
    }
}
</style>

<script>
export default {
    name: "ctl",
    data: () => ({
        config:   huds.config(),
        debug:    typeof huds.options.debug === "boolean" ? huds.options.debug : false
    }),
    components: {
        "popup":        Vue.loadComponent("ctl-widget-popup.vue")
    },
    created () {
        /*  receive messages for the attendance channel  */
        huds.bind("attendance", (event, data) => {
            /*  just react on correctly structured messages  */
            if (!(   typeof data.client  === "string" && data.client !== ""
                  && typeof data.event   === "string" && data.event  !== ""))
                return
            const a1 = this.$refs.attendance
            a1.attendance(data)
            const a2 = this.$refs.attendees
            a2.attendance(data)
            const f = this.$refs.feeling
            f.attendance(data)
            const p = this.$refs.popup
            p.attendance(data)
        })

        /*  receive messages for the attendance channel  */
        huds.bind("feeling", (event, data) => {
            /*  just react on correctly structured messages  */
            if (!(   typeof data.client    === "string" && data.client !== ""
                  && typeof data.challenge === "number" && typeof data.mood === "number"))
                return
            const f = this.$refs.feeling
            f.event(data)
        })

        /*  receive messages for the progress and agenda channel  */
        huds.bind("progress.*", (event, data) => {
            const t = this.$refs.timer
            const a = this.$refs.agenda
            if (event === "progress.prev") {
                a.prev()
                t.restart()
            }
            else if (event === "progress.next") {
                a.next()
                t.restart()
            }
        })

        /*  receive messages for the popup channel  */
        huds.bind("popup.add", (event, data) => {
            const a = this.$refs.popup
            a.add(data)
        })
        huds.bind("popup.remove", (event, data) => {
            const a = this.$refs.popup
            a.remove()
        })

        /*  receive messages for the voting channel  */
        let votesEnabled = false
        huds.bind("votes.*", (event, data) => {
            if (event === "votes.toggle")
                votesEnabled = !votesEnabled
        })

        /*  receive messages from a companion chat  */
        huds.bind("message", (event, data) => {
            /*  just react on correctly structured messages  */
            if (!(   (typeof data.client === "string" && data.client !== "")
                  && (typeof data.text === "string")))
                return

            /*  filter message markup  */
            data.text = data.text
                .replace(/&nbsp;/g, " ")
                .replace(/\s+/g, " ")
                .replace(/^\s+/, "")
                .replace(/\s+$/, "")
            data.text = data.text.replace(/<([a-z][a-zA-Z0-9:-]*)(?:\s+="[^""]*")*\s*>(.*?)<\/\1>/g, (_, tag, body) => {
                if (tag.match(/^(?:strong|em|u|s|b|i)$/))
                    return `<${tag}>${body}</${tag}>`
                else
                    return body
            })

            /*  short-circuit voting messages  */
            if (votesEnabled)
                return

            /*  react on particular message types  */
            if (data.text.match(/^(.+?)\?$/)) {
                const a = this.$refs.popup
                a.add({ ...data, type: "question" })
            }
            else if (data.text.match(/^(.+?)!$/)) {
                const a = this.$refs.popup
                a.add({ ...data, type: "objection" })
            }
            else {
                const a = this.$refs.popup
                a.add({ ...data, type: "comment" })
            }
        })
    }
}
</script>

