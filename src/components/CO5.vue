<template>
  <div>
  </div>
</template>

<script>
export default {
  name: 'CO5',
  data () {
    return {
      notes: ['C', 'C♯', 'D', 'D♯', 'E', 'F', 'F♯', 'G', 'G♯', 'A', 'A♯', 'B'],
      // frank can go down and eat breakfast
      magic: ['F', 'C', 'G', 'D', 'A', 'E', 'B'],
      accidents: ['♯', '♭', '#', 'b'],
      key: 'C'
    }
  },
  props: {
    startNote: {
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
    isMajor() {

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
    parse(note) {
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
    },

  },
  computed: {
    cycle() {
      let cycle = {"Major": [this.key], "Minor": [this.prev(this.key, 3)]}
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
  },
  mounted() {
    this.key = this.parse(this.startNote)
    console.log(this.cycle)
  },
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
