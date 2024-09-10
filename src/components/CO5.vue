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
    console.log(this.scale)
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
