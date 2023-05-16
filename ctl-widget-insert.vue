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
  <div class="tabs-level-1">
    <!--  ==== MONITOR ====  -->
    <div class="header">Texteingabe</div>
    <div class="desc">
      Dieses <b>Textfeld</b> erlaubt es, Texte als Overlay in den Stream zu
      senden.
    </div>
    <div class="control">
      <div class="label1">Text Zeile 1:</div>
      <input
        class="text"
        ref="line1"
        v-model="line1"
        :maxlength="line1MaxLength"
        @input="countChars(1, line1)"
        v-on:keyup.enter="sendText"
      />
      <p>({{ line1Length }}/{{ line1MaxLength }})</p>
    </div>
    <div class="control">
      <div class="label1">Text Zeile 2:</div>
      <input
        class="text"
        ref="line2"
        v-model="line2"
        :maxlength="line2MaxLenght"
        @input="countChars(2, line2)"
        v-on:keyup.enter="sendText"
      />
      <p>({{ line2Length }}/{{ line2MaxLenght }})</p>
      <div
        class="button"
        :class="{ disabled: buttonDisabeld }"
        v-on:click="sendText"
      >
        Send
      </div>
    </div>
  </div>
</template>

<style>
@import "./app.css";
</style>

<script>
export default {
  name: "title-bar",
  props: {
    questionbackground: { type: String, default: "" },
    questiontitlecolor: { type: String, default: "" },
    questionmessagecolor: { type: String, default: "" },
    objectionbackground: { type: String, default: "" },
    objectiontitlecolor: { type: String, default: "" },
    objectionmessagecolor: { type: String, default: "" },
    commentbackground: { type: String, default: "" },
    commenttitlecolor: { type: String, default: "" },
    commentmessagecolor: { type: String, default: "" },
    privacylevel: { type: String, default: "" },
  },
  data: () => ({
    text: "",
    line1: "",
    line2: "",
    line1Length: 0,
    line2Length: 0,
    line1MaxLength: 320,
    line2MaxLenght: 160,
    buttonDisabeld: false,
    timeout:
      huds.config().insert.timeDelay +
      huds.config().insert.timeAnimation1 * 2 +
      huds.config().insert.timeAnimation2 * 2 +
      huds.config().insert.timeDuration +
      huds.config().insert.timePause,
    queue: [],
    popups: [],
    attendees: {},
    timer: null,
  }),
  computed: {
    style: HUDS.vueprop2cssvar(),
  },
  setup() {
    const boxes = Vue.ref([]);
    Vue.onBeforeUpdate(() => {
      boxes.value = [];
    });
    return { boxes };
  },
  methods: {
    /*  add an insert */
    async sendText() {
      /* make one string from the two line input */
      if (!this.buttonDisabeld) {
        let txt;
        this.buttonDisabeld = true;

        if (this.line1Length > 0 && this.line2Length == 0) txt = this.line1;
        else if (this.line1Length == 0 && this.line2Length > 0)
          txt = this.line2;
        else if (this.line1Length > 0 && this.line2Length > 0)
          txt = this.line1 + "\n" + this.line2;
        else return;
        const data = { text: txt };

        huds.send("insert", data);
        //this.line1 = "";
        //this.line2 = "";
        //this.line1Length = 0;
        //this.line2Length = 0;
        setTimeout(() => (this.buttonDisabeld = false), this.timeout);
      }
    },

    async countChars(line, input) {
      if (line == 1) {
        this.line1Length = input.length;
      } else if (line == 2) {
        this.line2Length = input.length;
      }
    },
  },
};
</script>
