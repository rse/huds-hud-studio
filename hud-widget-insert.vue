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
    <div v-show="enabled" ref="lowerthird" class="lowerthird"
            v-bind:style="{ 'left' : config.insert.boxXpos + 'px',
                            'top'  : config.insert.boxYpos + 'px',
                            'width': config.insert.boxWidth + 'px',
                            'display': visible
                        }"
    >
        <div class="door" ref="door">
            <div class="bar" ref="bar"
                v-bind:style="{ 
                                'background': 'linear-gradient(180deg,' + config.insert.barColor1 + ',' 
                                                                                    + config.insert.barColor2 + ')',
                                'height':     2 + config.insert.line1Height + config.insert.line2Height + 4 + 'px' 
                              }"
            ></div>
        </div>
        <div class="content">
            <div class="line1" ref="line1"
                v-bind:style="{ 'line-height': config.insert.line1Height + 'px',
                                'color':       config.insert.line1Color1,
                                'text-shadow': config.insert.line1Shadow1,
                                'font-family': config.insert.line1FontFamily1,
                                'font-size'  : 0.75 * config.insert.line1Height + 'px',
                                'font-weight': config.insert.line1FontWeight1,
                                'font-style':  config.insert.line1FontStyle1,
                                'background':  config.insert.boxBackground,
                                'width':       config.insert.boxWidth - 30 - 6 + 'px', 
                              }"
            >
                <span class="text1">{{ text }}</span>
            </div>
            <div class="line2" ref="line2"
                v-bind:style="{ 'line-height': config.insert.line2Height + 'px',
                                'color':       config.insert.line2Color1,
                                'text-shadow': config.insert.line2Shadow1,
                                'font-family': config.insert.line2FontFamily1,
                                'font-size'  : 0.75 * config.insert.line2Height + 'px',
                                'font-weight': config.insert.line2FontWeight1,
                                'font-style':  config.insert.line2FontStyle1,
                                'background':  config.insert.boxBackground,
                                'width':       config.insert.boxWidth - 30 - 6 + 'px', 
                                }"
            >
                <span class="text1">{{ text }}</span>
            </div>
        </div>
    </div>
</template>

<style lang="less" scoped>
@import "./lowerthird.css";
</style>

<script>
export default {
    name: "insert",
    props: {
    },
    data: () => ({
        text : "",
        config:   huds.config()
    }),
    computed: {
        style: HUDS.vueprop2cssvar()
    },
    methods: {
        /*  take the text, create the insert widget and visualize it through animation  */
        set (data) {
            this.text = data

            const def = huds.config().insert

            /*  grab the elements in the DOM fragment  */
            const elBar   = this.$refs.bar
            const elLine1 = this.$refs.line1
            const elLine2 = this.$refs.line2

            /* calculate width of text to assess the horizontal size of the textbox */
            const textwidth = this.measureText(this.text, def.line1Height * 0.75, elLine1.style).width

            /*  calculate start and end position for animation  */
            const barStartPos  = 1 + 2 + def.line1Height + def.line2Height + 4
            const barEndPos    = 0
            const lineStartPos = -textwidth - 30      // honor padding
            const lineEndPos   = 0

            /*  set init position for animated elements  */
            elBar.style.top    = barStartPos + "px"
            elLine1.style.left = lineStartPos + "px"
            elLine2.style.left = lineEndPos + "px"

            /*  create animation timeline  */
            const tl = anime.timeline({
                autoplay:  true,
                direction: "normal"
            })

            /*  coming: bar  */
            tl.add({
                targets:   elBar,
                delay:     def.timeDelay,
                duration:  def.timeAnimation1,
                easing:    "easeOutSine",
                top:       [ barStartPos, barEndPos ] 
            })

            /*  coming: line 1 and 2 */
            tl.add({
                targets:   [ elLine1, elLine2 ],
                duration:  def.timeAnimation2,
                endDelay:  def.timeDuration,
                easing:    "easeOutSine",
                left:      [ lineStartPos, lineEndPos ]
            })

            /*  going: line 1 and 2  */
            tl.add({
                targets:   [ elLine1, elLine2 ],
                duration:  def.timeAnimation2,
                easing:    "easeInSine",
                left:      [ lineEndPos, lineStartPos ] 
            })

            /*  going: bar  */
            tl.add({
                targets:   elBar,
                duration:  def.timeAnimation1,
                easing:    "easeInSine",
                top:       [ barEndPos, barStartPos ]
            })
        },
        
        /* calculate the width and height of a text string based on font and font size */
        /* (using Tofandel#s solution in https://stackoverflow.com/questions/118241/calculate-text-width-with-javascript) */
        measureText(pText, pFontSize, pStyle) {
            var lDiv = document.createElement('div');

            document.body.appendChild(lDiv);

            if (pStyle != null) {
                lDiv.style = pStyle;
            }
            lDiv.style.fontSize = "" + pFontSize + "px";
            lDiv.style.position = "absolute";
            lDiv.style.left = -1000;
            lDiv.style.top = -1000;

            lDiv.textContent = pText;

            var lResult = {
                width: lDiv.clientWidth,
                height: lDiv.clientHeight
            };

            document.body.removeChild(lDiv);
            lDiv = null;

            return lResult;
        }
    }
}
</script>