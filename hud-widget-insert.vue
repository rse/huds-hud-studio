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
                                'height':     2 + config.insert.lineHeight + 4 + 'px' 
                              }"
            ></div>
        </div>
        <div class="content">
            <div class="line" ref="line"
                v-bind:style="{ 'height': config.insert.lineHeight + 'px',
                                'color':       config.insert.lineColor,
                                'text-shadow': config.insert.lineShadow,
                                'font-family': config.insert.lineFontFamily,
                                'font-size'  : 0.375 * config.insert.lineHeight + 'px', // want to have two lines
                                'font-weight': config.insert.lineFontWeight,
                                'font-style':  config.insert.lineFontStyle,
                                'background':  config.insert.boxBackground,
                                'width':       config.insert.boxWidth - 30 - 6 + 'px', 
                                'margin-right': 100 + 'px'
                              }"
            >
                <span class="text">{{ text }}</span>
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
            //this.text = "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna"

            const def = huds.config().insert

            /*  grab the elements in the DOM fragment  */
            const elBar  = this.$refs.bar
            const elLine = this.$refs.line

            /* calculate width of text to assess the horizontal size of the textbox */
            const textwidth = this.measureText(this.text, def.lineHeight * 0.375, elLine.style).width

            /*  calculate start and end position for animation  */
            const barStartPos  = 1 + 2 + def.lineHeight + 4
            const barEndPos    = 0
            const lineStartPos = -textwidth - 30 - 6      // honor padding
            const lineEndPos   = 0

            /*  set init position for animated elements  */
            elBar.style.top   = barStartPos + "px"
            elLine.style.left = lineStartPos + "px"

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

            /*  coming: line 1 */
            tl.add({
                targets:   elLine,
                duration:  def.timeAnimation2,
                endDelay:  def.timeDuration,
                easing:    "easeOutSine",
                left:      [ lineStartPos, lineEndPos ]
            })

            /*  going: line 1 */
            tl.add({
                targets:   elLine,
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