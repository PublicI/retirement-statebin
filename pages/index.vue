<template>
    <no-ssr>
        <div class="statebinContainer">
            <svg>
            </svg>

            <div class="statebins">
                <div :style="'top:' + bin.y + 'px;left:' + bin.x + 'px;background-color:' + bin.color" class="statebin" v-for="bin in bins" v-tooltip="{ content: '<b>' + bin.name + '</b><br>' + bin.percent + ' percent<br>' + bin.number + ' workers' }">
                    {{bin.abbrev}}
                </div>
            </div>

            <p class="source">Source: AARP analysis of U.S. Census Bureau data</p>
        </div>
    </no-ssr>
</template>

<script>
import stats from '~/assets/stats.csv';
import { intword, intcomma, postal } from 'journalize';
import * as d3 from 'd3';
import { legendColor } from 'd3-svg-legend';

export default {
    data() {
        return {
            stats,
            grid: ['                                  ME',
                '                   WI          VT NH',
                '    WA ID MT ND MN IL MI    NY MA',
                '    OR NV WY SD IA IN OH PA NJ CT RI',
                '    CA UT CO NE MO KY WV VA MD DE',
                '       AZ NM KS AR TN NC SC',
                '             OK LA MS AL GA',
                '    HI AK    TX             FL']
        };
    },
    mounted() {
        let svg = d3.select('.statebinContainer svg');

        svg.append('g')
            .attr('class', 'legendLinear')
            .attr('transform', 'translate(20,20)');

        let scale = d3.scaleLinear()
            .domain([30, 70])
            .range([0, 1]);

        let legendLinear = legendColor()
            .shapeWidth(30)
            .cells([30, 40, 50, 60, 70])
            .orient('horizontal')
            .scale(scale);

        svg.select('.legendLinear')
            .call(legendLinear);
    },
    computed: {
        bins() {
            let binsRef = {};
            let bins = [];

            // let w = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);

            let boxSize = 26;

            /*
            if (w > 500 && w <= 541) {
                boxSize = 22;
            }
            */

            let re = /\w+/g;

            let scale = d3.scaleLinear()
                .domain([30, 70])
                .range([0, 1]);

            this.grid.forEach(function(line, i) {
                let m;
                while (m = re.exec(line)) { // eslint-disable-line no-cond-assign
                    let state = {
                        abbrev: m[0],
                        x: (m.index / 3)*boxSize-(boxSize+2),
                        y: i*boxSize,
                        color: null,
                        name: null,
                        percent: null,
                        number: null
                    };

                    bins.push(state);

                    binsRef[state.abbrev] = state;
                }
            });

            this.stats.forEach(function (d) {
                let abbrev = postal(d.state);
                if (abbrev in binsRef) {
                    let count = parseInt(d.number);

                    binsRef[abbrev].color = d3.interpolateYlOrBr(scale(parseInt(d.percent)));
                    binsRef[abbrev].name = d.state;
                    binsRef[abbrev].percent = parseInt(d.percent);
                    binsRef[abbrev].number = count >= 1000000 ? intword(count) : intcomma(count);
                }
            });

            return bins;
        }
    }
};
</script>

<style>
.statebins {
    position: relative;
    width: 300px;
    height: 220px;
    /* margin-top: -19px; */
}

.statebin {
    position: absolute;
    width: 24px;
    height: 24px;
    background-color: #eee;
    color: white;
    text-align: center;
    font-size: 12px;
    line-height: 20px;
    padding-top: 1px;
}

.tooltip {
    display: block !important;
    z-index: 10000;
}

.tooltip .tooltip-inner {
    background: white;
    color: black;
    border-radius: 2px;
    padding: 5px 10px 4px;
    font-family: tablet-gothic-n2,tablet-gothic,Helvetica Neue,Helvetica,Arial,sans-serif;
    font-size: 16px;
    line-height: 19px;
}

.tooltip .tooltip-arrow {
    width: 0;
    height: 0;
    border-style: solid;
    position: absolute;
    margin: 5px;
    border-color: white;
    z-index: 1;
}

.tooltip[x-placement^="top"] {
    margin-bottom: 5px;
}

.tooltip[x-placement^="top"] .tooltip-arrow {
    border-width: 5px 5px 0 5px;
    border-left-color: transparent !important;
    border-right-color: transparent !important;
    border-bottom-color: transparent !important;
    bottom: -5px;
    left: calc(50% - 5px);
    margin-top: 0;
    margin-bottom: 0;
}

.tooltip[x-placement^="bottom"] {
    margin-top: 5px;
}

.tooltip[x-placement^="bottom"] .tooltip-arrow {
    border-width: 0 5px 5px 5px;
    border-left-color: transparent !important;
    border-right-color: transparent !important;
    border-top-color: transparent !important;
    top: -5px;
    left: calc(50% - 5px);
    margin-top: 0;
    margin-bottom: 0;
}

.tooltip[x-placement^="right"] {
    margin-left: 5px;
}

.tooltip[x-placement^="right"] .tooltip-arrow {
    border-width: 5px 5px 5px 0;
    border-left-color: transparent !important;
    border-top-color: transparent !important;
    border-bottom-color: transparent !important;
    left: -5px;
    top: calc(50% - 5px);
    margin-left: 0;
    margin-right: 0;
}

.tooltip[x-placement^="left"] {
    margin-right: 5px;
}

.tooltip[x-placement^="left"] .tooltip-arrow {
    border-width: 5px 0 5px 5px;
    border-top-color: transparent !important;
    border-right-color: transparent !important;
    border-bottom-color: transparent !important;
    right: -5px;
    top: calc(50% - 5px);
    margin-left: 0;
    margin-right: 0;
}

.tooltip.popover .popover-inner {
    background: #f9f9f9;
    color: black;
    padding: 24px;
    border-radius: 2px;
    box-shadow: 0 5px 30px rgba(black, .1);
}

.tooltip.popover .popover-arrow {
    border-color: #f9f9f9;
}

.tooltip[aria-hidden='true'] {
    visibility: hidden;
    opacity: 0;
    transition: opacity .15s, visibility .15s;
}

.tooltip[aria-hidden='false'] {
    visibility: visible;
    opacity: 1;
    transition: opacity .15s;
}
.source {
    font-size: 14px;
    line-height: 16px;
    color: #666;
    margin-bottom: 15px;
}
</style>
