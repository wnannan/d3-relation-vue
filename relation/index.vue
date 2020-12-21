<template>
  <div>
    <svg
      id="svg-content"
      ref="svg"
      tabindex="0"
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
    >
      <g />
      <rect />
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
export default {
  name: 'Relation',
  components: {},
  props: {},
  data() {
    return {
      links: [
        { source: 'a', target: 'b' },
        { source: 'b', target: 'e' },
        { source: 'c', target: 'f' },
        { source: 'c', target: 'g' },
        { source: 'd', target: 'h' },
        { source: 'a', target: 'c' },
        { source: 'a', target: 'd' },
        { source: 'h', target: 'i' },
      ],
      data: [],
      width: 1280,
      height: 600
    }
  },
  computed: {

  },
  watch: {

  },
  mounted() {
    this.create()
  },
  methods: {
    create() {
      this.data = {
        nodes: Array.from(new Set(this.links.flatMap(l => [l.source, l.target])), id => ({ id })),
        links: this.links
      }
      const svg = d3.select(this.$refs.svg)
        .attr('width', this.width)
        .attr('height', this.height)
      this.appendNode(svg)
      console.log(this.data)
    },
    appendNode(enter) {
      const links = this.data.links
      const nodes = this.data.nodes
      const that = this
      console.log(links)

      // 布局？
      const simulation = d3.forceSimulation()
        .nodes(nodes)
        .force('link', d3.forceLink(links).id(d => d.id).distance(150))
        .force('charge', d3.forceManyBody().strength(-140))
        .force('center', d3.forceCenter(this.width / 2, this.height / 2))
      const dragstart = function() {
        // d3.select(this).classed('fixed', true)
      }
      const dragged = function(d) {
        const event = d3.event
        d.fx = that.clamp(event.x, 0, that.width)
        d.fy = that.clamp(event.y, 0, that.height)
        simulation.alpha(1).restart()
      }
      const click = function(d) {
        const event = d3.event
        delete d.fx
        delete d.fy
        // d3.select(this).classed('fixed', false)
        simulation.alpha(1).restart()
      }
      // 箭头
      const markerPath = enter.append('defs')
        .append('marker')
        .attr('id', 'arrow')
        .attr('viewBox', '0 0 10 10')
        .attr('refX', 6)
        .attr('refY', 6)
        .attr('markerWidth', 6)
        .attr('markerHeight', 10)
        .attr('orient', 'auto')
        .append('path')
        .attr('fill', '#ccc')
        .attr('d', 'M0,0L-5,15L10,5')
        // 线条
      const link = enter.append('g')
        .attr('stroke', '#ccc')
        .attr('stroke-opacity', 0.6)
        .selectAll('path')
        .data(links)
        .join('path')
        .attr('id', d => `path_${d.index}`)
        .attr('fill', 'none')
        .attr('d', this.path)
        .attr('stroke', '#ccc')
        .attr('stroke-width', 2)
        // .attr('marker-end', 'url(#arrow)')
      // 节点
      const g = enter.append('g')
        .attr('fill', 'currentColor')
        .selectAll('g')
        .data(nodes)
        .join('g')
        .attr('id', d => `nodeDiv_${d.index}`)
        .classed('fixed', d => d.fx !== undefined)
        .call(d3
          .drag()
          .on('start', dragstart)
          .on('drag', dragged))
        .on('click', click)
      // 文字
      g.append('foreignObject')
        .append('xhtml:div')
        .attr('class', 'nodebox')
        .text(d => d.id)
      // 计算节点和线条位置
      simulation.on('tick', () => {
        g.attr('transform', d => `translate(${this.clamp(d.x, 0, this.width)},${this.clamp(d.y, 0, this.height)})`)
        link.attr('d', this.path)
      })
    },
    path(d) {
      const sourceX = this.clamp(d.source.x + 35, 0, this.width)
      const sourceY = this.clamp(d.source.y + 35, 0, this.height)
      const targetX = this.clamp(d.target.x + 35, 0, this.width)
      const targetY = this.clamp(d.target.y + 35, 0, this.height)
      return `M${sourceX},${sourceY} L${targetX},${targetY}`
    },
    clamp(x, lo, hi) {
      hi = hi - 70
      return x < lo ? lo : x > hi ? hi : x
    }
  }
}
</script>

<style lang="scss">
svg{
  width: 100%;
  height: 100%;
  font-size: 12px;
  outline: none;
  foreignObject{
    width: 70px;
    height: 70px;
  }
  .nodebox{
    width: 70px;
    height: 70px;
    line-height: 70px;
    border-radius: 50%;
    border: 2px solid #ccc;
    background: #eee;
    text-align: center;
  }
}

</style>
