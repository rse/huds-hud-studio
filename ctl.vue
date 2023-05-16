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
  <div class="app-control">
    <!--  HEADER  -->
    <div class="head">HUDS Studio Control</div>

    <!--  BODY  -->
    <div class="body">
      <div class="ctl">
        <insert ref="insert" class="row"></insert>
        <logo ref="logo" class="row"></logo>
      </div>
    </div>

    <!--  FOOTER  -->
    <div class="foot"></div>
  </div>
</template>

<style>
@import "./app.css";
</style>

<script>
export default {
  name: "ctl",
  data: () => ({
    config: huds.config(),
    debug: typeof huds.options.debug === "boolean" ? huds.options.debug : false,
  }),
  components: {
    insert: Vue.loadComponent("ctl-widget-insert.vue"),
    logo: Vue.loadComponent("ctl-widget-logo.vue"),
  },
  created() {
    huds.bind("insert.toggle", (event, data) => {
      const a = this.$refs.insert;
      if (!(typeof data.text === "string" && data.text !== "")) return;
      a.set(data.text);
      a.toggle();
    });

    /*  receive messages for the attendance channel  */
    huds.bind("attendance", (event, data) => {
      /*  just react on correctly structured messages  */
      if (
        !(
          typeof data.client === "string" &&
          data.client !== "" &&
          typeof data.event === "string" &&
          data.event !== ""
        )
      )
        return;
      const a1 = this.$refs.attendance;
      a1.attendance(data);
      const a2 = this.$refs.attendees;
      a2.attendance(data);
      const f = this.$refs.feeling;
      f.attendance(data);
      const p = this.$refs.popup;
      p.attendance(data);
    });

    /*  receive messages for the attendance channel  */
    huds.bind("feeling", (event, data) => {
      /*  just react on correctly structured messages  */
      if (
        !(
          typeof data.client === "string" &&
          data.client !== "" &&
          typeof data.challenge === "number" &&
          typeof data.mood === "number"
        )
      )
        return;
      const f = this.$refs.feeling;
      f.event(data);
    });

    /*  receive messages for the progress and agenda channel  */
    huds.bind("progress.*", (event, data) => {
      const t = this.$refs.timer;
      const a = this.$refs.agenda;
      if (event === "progress.prev") {
        a.prev();
        t.restart();
      } else if (event === "progress.next") {
        a.next();
        t.restart();
      }
    });

    /*  receive messages for the popup channel  */
    huds.bind("popup.add", (event, data) => {
      const a = this.$refs.popup;
      a.add(data);
    });
    huds.bind("popup.remove", (event, data) => {
      const a = this.$refs.popup;
      a.remove();
    });

    /*  receive messages for the voting channel  */
    let votesEnabled = false;
    huds.bind("votes.*", (event, data) => {
      if (event === "votes.toggle") votesEnabled = !votesEnabled;
    });

    /*  receive messages from a companion chat  */
    huds.bind("message", (event, data) => {
      /*  just react on correctly structured messages  */
      if (
        !(
          typeof data.client === "string" &&
          data.client !== "" &&
          typeof data.text === "string"
        )
      )
        return;

      /*  filter message markup  */
      data.text = data.text
        .replace(/&nbsp;/g, " ")
        .replace(/\s+/g, " ")
        .replace(/^\s+/, "")
        .replace(/\s+$/, "");
      data.text = data.text.replace(
        /<([a-z][a-zA-Z0-9:-]*)(?:\s+="[^""]*")*\s*>(.*?)<\/\1>/g,
        (_, tag, body) => {
          if (tag.match(/^(?:strong|em|u|s|b|i)$/))
            return `<${tag}>${body}</${tag}>`;
          else return body;
        }
      );

      /*  short-circuit voting messages  */
      if (votesEnabled) return;

      /*  react on particular message types  */
      if (data.text.match(/^(.+?)\?$/)) {
        const a = this.$refs.popup;
        a.add({ ...data, type: "question" });
      } else if (data.text.match(/^(.+?)!$/)) {
        const a = this.$refs.popup;
        a.add({ ...data, type: "objection" });
      } else {
        const a = this.$refs.popup;
        a.add({ ...data, type: "comment" });
      }
    });
  },
};
</script>
