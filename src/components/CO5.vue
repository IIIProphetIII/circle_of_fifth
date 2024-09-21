<template>
  <div ref="container">
  </div>
</template>

<script>
import * as d3 from 'd3'
export default {
  name: 'CO5',
  data () {
    return {
      notes: ['C', 'C♯', 'D', 'D♯', 'E', 'F', 'F♯', 'G', 'G♯', 'A', 'A♯', 'B'],
      // frank can go down and eat breakfast
      magic: ['F', 'C', 'G', 'D', 'A', 'E', 'B'],
      interval: [2, 2, 1, 2, 2, 2, 1],
      accidents: ['♯', '♭', '#', 'b'],
    }
  },
  props: {
    StartNote: {
      type: String,
      default: 'C'
    },
    Key:{
      type: String,
      default: 'C'
    }
  },
  methods: {
    next(note, interval = 1) {
      return this.notes[(this.notes.indexOf(note) + interval) % 12];
    },
    prev(note, interval = 1) {
      return this.notes[(this.notes.indexOf(note) - interval + 12) % 12];
    },
    isSharp(note) {
      if (note.length === 2 && this.accidents.includes(note[1])) {
        if (note[1] === '#' || note[1] === '♯') {
          return true
        }
      }
      return false
    },
    switchAccident(note) {
      if (note.length === 2 && this.accidents.includes(note[1])) {
        if (this.isSharp(note)) {
          let base = this.next(note)
          return `${base}♭`
        }else{
          let base = this.prev(note[0])
          return `${base[0]}♯`
        }
      }
      return note
    },
    enableUI(){
      const width = window.innerWidth/2
      const height = Math.min(width, window.innerHeight);
      const radius = Math.min(width, height) / 2;
      const arcMajor = d3.arc().innerRadius(radius * 0.75).outerRadius(radius - 1).padAngle(0.02)
      const arcMinor = d3.arc().innerRadius(radius * 0.50).outerRadius(radius * 0.75).padAngle(0.03)
      const pie = d3.pie().sort(null).value(1).padAngle(1 / radius)
      const rotationAngle = Math.PI / this.cycle.Major.length
      const svg = d3.create('svg')
        .attr('height', height)
        .attr('width', width)
        .attr('viewBox', [-width / 2, -height / 2, width, height])
        .style('style', 'max-width: 100%; height: auto;')

      svg.append('g')
        .selectAll('outer')
        .data(pie(this.cycle.Major))
        .join("path")
        .attr("fill", '#72ac51').attr("d", (d)=>arcMajor({
          startAngle: d.startAngle - rotationAngle,
          endAngle: d.endAngle - rotationAngle
        }))
      svg.append('g')
        .selectAll('outer')
        .data(pie(this.cycle.Major))
        .join("text")
        .attr("transform", (d) => `translate(${arcMajor.centroid({
            startAngle: d.startAngle - rotationAngle,
            endAngle: d.endAngle - rotationAngle
          })})`)
        .attr("class", 'outer')
        .text((d)=>d.data)

      svg.append('g')
        .selectAll('inner')
        .data(pie(this.cycle.Minor))
        .join("path")
        .attr("fill", '#d4e6ca').attr("d", (d)=>arcMinor({
          startAngle: d.startAngle - rotationAngle,
          endAngle: d.endAngle - rotationAngle
        }))
      svg.append('g').selectAll('inner').data(pie(this.cycle.Minor)).join("text")
        .attr("transform", (d) => `translate(${arcMinor.centroid({
          startAngle: d.startAngle - rotationAngle,
          endAngle: d.endAngle - rotationAngle
        })})`)
        .attr("class", 'inner')
        .text((d)=>d.data)

      this.$refs.container.append(svg.node());
    },
  },
  computed: {
    startNote(){
      let note = this.StartNote
      if (typeof note === 'number' && note >= 0 && note < this.notes.length ) {
        return this.notes[note]
      }
      if (typeof note === 'string' && note.length > 0 && note.length <= 2) {
        if (note.length === 1) {
          return note.toUpperCase()
        }
        if (note.length === 2) {
          if (this.notes.includes(note[0].toUpperCase()) && this.accidents.includes(note[1])) {
            return `${note[0].toUpperCase()}${note[1].toLowerCase()}`
          }
        }
      }
      console.log('Invalid note: ' + note)
      return 'C'      
    },

    key(){
      let note = this.Key
      if (typeof note === 'string' && note.length > 0 && note.length <= 2) {
        if (note.length === 1 && this.notes.includes(note.toUpperCase())){
          return note
        }
        if (note.length === 2) {
          if (this.notes.includes(note[0].toUpperCase()) && this.accidents.includes(note[1])) {
            return `${note[0]}${note[1].toLowerCase()}`
          }
        }
      }
      console.log('Invalid Key: ' + note)
      return 'C'
    },

    cycle() {
      let cycle = {"Major": [this.startNote], "Minor": [this.prev(this.startNote, 3)]}
      for (let index = 0; index < 11; index++) {
        let prev = cycle["Major"].slice(-1)[0]
        cycle["Major"].push(this.next(prev,7))
        let cur = cycle["Major"].slice(-1)[0]
        cycle["Minor"].push(this.prev(cur, 3))
      }
      let majorMid = cycle["Major"][cycle["Major"].length/2] 
      cycle["Major"][cycle["Major"].length/2] = this.switchAccident(majorMid) +"/"+majorMid
      let minorMid = cycle["Minor"][cycle["Minor"].length/2]
      cycle["Minor"][cycle["Minor"].length/2] = this.switchAccident(minorMid) +"/"+minorMid
      for (let index = cycle["Major"].length-1; index > cycle["Major"].length/2 ; index--) {
        cycle["Major"][index] = this.switchAccident(cycle["Major"][index])
        cycle["Minor"][index] = this.switchAccident(cycle["Minor"][index])
      }
      return cycle
    },

    isMajor() {
      return (this.key === this.key.toUpperCase())?true:false
    },

    scale(){
      let scale = {}
      if (this.isMajor) {
        scale = {"Major": [this.key], "Minor": [this.prev(this.key, 3).toLowerCase()]}
      }else{
        scale = {"Major": [this.next(this.key.toUpperCase(), 3)], "Minor": [this.key]}
      }
      for (let index = 0; index < 6; index++) {
        let prev = scale["Major"].slice(-1)[0]
        let cur = this.next(prev, this.interval[index])
        scale["Major"].push(cur)
        scale["Minor"].push(this.prev(cur, 3).toLowerCase())
      }
      return scale
    }

  },
  mounted() {
    console.log(this.cycle)
    this.enableUI()
  },
}
</script>

<style scope>
.outer {
  text-anchor: middle;
  font-weight: bold;
  fill: #ffffff;
  font-size: 25px;
}
.inner {
  text-anchor: middle;
  fill: #555555;
  font-size: 20px;
}
</style>
