<template>
  <div>
    <svg
      id="svg-content"
      ref="svg"
      tabindex="0"
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
    />
  </div>
</template>

<script>
import * as d3 from 'd3'
import json from './module/flare-2.json'
export default {
  name: 'CollapsibleTree',
  components: {},
  props: {},
  data() {
    return {
      data: [],
      root: {},
      width: 1280,
      height: 600,
      dx: 20,
      dy: 1280 / 6,
      margin: {
        top: 20,
        right: 120,
        bottom: 20,
        left: 60
      },
      colors: ['#25a4f6', '#92d1fa', '#a2d403', '#f7881f', '#f7881f'],
      svg: null,
      gLink: null,
      gNode: null
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
      const { dx, dy, margin, width, height } = this
      const root = d3.hierarchy(json)
      root.x0 = this.dy / 2
      root.y0 = 0
      root.descendants().forEach((d, i) => {
        d.id = i
        d._children = d.children
        if (d.depth && d.data.name.length !== 7) d.children = null
      })
      this.data = root
      const svg = d3.select(this.$refs.svg)
        .attr('viewBox', [-margin.left, -margin.top, width, height])
        .style('font', '12px sans-serif')
        .style('user-select', 'none')
      const gLink = svg.append('g')
        .attr('fill', 'none')
        // .attr('stroke', '#555')
        // .attr('stroke-opacity', 0.4)
        // .attr('stroke-width', 1.5)

      const gNode = svg.append('g')
        .attr('cursor', 'pointer')
        .attr('pointer-events', 'all')
      this.svg = svg
      this.gLink = gLink
      this.gNode = gNode
      this.update(root)
      return svg.node()
    },
    update(source) {
      const { dx, dy, margin, width, svg, gLink, gNode, data, colors } = this
      const diagonal = d3.linkHorizontal().x(d => d.y).y(d => d.x) // 横向贝塞尔曲线
      const tree = d3.tree().nodeSize([dx, dy]) // 每个节点大小
      const root = data
      const duration = d3.event && d3.event.altKey ? 2500 : 250
      const nodes = root.descendants().reverse()
      const links = root.links()
      tree(data)
      let left = root
      let right = root
      root.eachBefore(node => {
        if (node.x < left.x) left = node
        if (node.x > right.x) right = node
      })
      const height = right.x - left.x + margin.top + margin.bottom

      const transition = svg.transition()
        .duration(duration) // 过渡持续时间
        .attr('viewBox', [-margin.left, left.x - margin.top, width, height]) // min-x, min-y, width,height
        .tween('resize', window.ResizeObserver ? null : () => () => svg.dispatch('toggle'))

      // Update the nodes…
      const node = gNode.selectAll('g')
        .data(nodes, d => d.id)

      // Enter any new nodes at the parent's previous position.
      const nodeEnter = node.enter().append('g')
        .attr('transform', d => `translate(${source.y0},${source.x0})`)
        .attr('fill-opacity', 0)
        .attr('stroke-opacity', 0)
        .on('click', (d) => {
          d.children = d.children ? null : d._children
          this.update(d)
        })

      nodeEnter.append('circle')
        .attr('r', 3)
        .attr('fill', d => d._children ? colors[d.depth] : '#999')
        // .attr('stroke-width', 10)

      nodeEnter.append('text')
        .attr('dy', '3px')
        .attr('x', d => d._children ? -6 : 6)
        .attr('text-anchor', d => d._children ? 'end' : 'start')
        .text(d => d.data.name)
        // .clone(true).lower()
        .attr('stroke-linejoin', 'round')
        .attr('stroke-width', 3)
        // .attr('stroke', 'white')

      // Transition nodes to their new position.
      const nodeUpdate = node.merge(nodeEnter).transition(transition)
        .attr('transform', d => `translate(${d.y},${d.x})`)
        .attr('fill-opacity', 1)
        .attr('stroke-opacity', 1)

      // Transition exiting nodes to the parent's new position.
      const nodeExit = node.exit().transition(transition).remove()
        .attr('transform', d => `translate(${source.y},${source.x})`)
        .attr('fill-opacity', 0)
        .attr('stroke-opacity', 0)

      // Update the links…
      const link = gLink.selectAll('path')
        .data(links, d => d.target.id)

      // Enter any new links at the parent's previous position.
      const linkEnter = link.enter().append('path')
        .attr('d', d => {
          const o = { x: source.x0, y: source.y0 }
          return diagonal({ source: o, target: o })
        })
        .style('stroke', function(d) {
          return colors[d.source.depth]
        })

      // Transition links to their new position.
      link.merge(linkEnter).transition(transition)
        .attr('d', diagonal)

      // Transition exiting nodes to the parent's new position.
      link.exit().transition(transition).remove()
        .attr('d', d => {
          const o = { x: source.x, y: source.y }
          return diagonal({ source: o, target: o })
        })

      // Stash the old positions for transition.
      root.eachBefore(d => {
        d.x0 = d.x
        d.y0 = d.y
      })
    },
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
