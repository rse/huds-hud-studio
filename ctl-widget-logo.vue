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
    <div class="header">Logo</div>
    <div class="desc">
      Logo ein- oder ausblenden.
    </div>
    <div class="control">
      <div class="label1">Show Logo:</div>
      <label class="switch">
        <input type="checkbox" v-model="this.logoVisible" v-on:click="toggleLogo">
        <span class="slider round"></span>
      </label>
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
    questionbackground:    { type: String, default: "" },
    questiontitlecolor:    { type: String, default: "" },
    questionmessagecolor:  { type: String, default: "" },
    objectionbackground:   { type: String, default: "" },
    objectiontitlecolor:   { type: String, default: "" },
    objectionmessagecolor: { type: String, default: "" },
    commentbackground:     { type: String, default: "" },
    commenttitlecolor:     { type: String, default: "" },
    commentmessagecolor:   { type: String, default: "" },
    privacylevel:          { type: String, default: "" },
  },
  data: () => ({
    logoVisible: true,
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
    async toggleLogo() {
      /* toggle logo visbile*/
      huds.send("logo", !this.logoVisible);
    }
  },
};
</script>
