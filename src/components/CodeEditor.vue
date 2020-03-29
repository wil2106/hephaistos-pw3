<template>
  <div :id="'editor-' + name"
    class="exercise-editor-ace-editor"
    :style='{ height: editorHeight }'>
  </div>
</template>

<script>
import ace from 'ace-builds/src-noconflict/ace'
import 'ace-builds/src-noconflict/theme-monokai'
import 'ace-builds/src-noconflict/mode-python'
import 'ace-builds/src-noconflict/mode-javascript'
import 'ace-builds/src-noconflict/mode-c_cpp'
import 'ace-builds/webpack-resolver'

export default {
  props: {
    lang: String,
    // content: String,
    regionData: Object,
    name: String,
    editorHeight: {
      type: String,
      default: '20rem'
    }
  },
  data: () => ({
    editor: null,
    contentJustChanged: false,
    readOnlyLines: []
  }),
  watch: {
    editorHeight () {
      this.editor.resize()
    },
    regionData (newVal) {
      // fill regions content
      this.contentJustChanged = true
      this.editor.setValue(newVal.regions.join(''))
      this.editor.clearSelection()

      // remove previous highlights
      for (var id in this.editor.session.$backMarkers) {
        this.editor.session.removeMarker(id)
      }
      // highlighting regions
      var Range = ace.require('ace/range').Range
      var cursorPosition = 0
      this.readOnlyLines = []
      for (let regionNum = 0; regionNum < this.regionData.regions.length; regionNum++) {
        var lines = this.regionData.regions[regionNum].split('\n')
        var markerName = ''
        switch (this.regionData.templateRegions[regionNum]) {
          case 6:
            markerName = 'blueMarker'
            break
          case 4:
            markerName = 'greenMarker'
            break
          case 0:
            markerName = 'greyMarker'
            break
          default:
            markerName = ''
        }
        for (let lineNum = 0; lineNum < lines.length - 1; lineNum++) {
          if (markerName.length > 0) {
            this.editor.session.addMarker(new Range(cursorPosition, 0, cursorPosition, 1), markerName, 'fullLine')
            if (markerName === 'greenMarker') {
              this.readOnlyLines.push(cursorPosition + 1)
            }
          }
          cursorPosition += 1
        }
      }
      this.editor.commands.on('exec', (e) => {
        this.onInsert(e)
      })
    },
    lang (newVal) {
      if (this.editor) {
        this.changeLang(newVal)
      }
    }
  },
  mounted () {
    this.editor = ace.edit('editor-' + this.name)
    this.editor.setTheme('ace/theme/monokai')
    this.editor.setFontSize('1.1rem')
    this.changeLang(this.lang)
  },
  beforeDestroy () {
    if (this.editor) {
      this.editor.destroy()
      this.editor.container.remove()
    }
  },
  methods: {
    changeLang (lang) {
      if (typeof lang === 'undefined') {
        lang = 'python'
      } else if (lang === 'c') {
        lang = 'c_cpp'
      }
      this.editor.session.setMode(`ace/mode/${lang}`)
    },
    onInsert (event) {
      var rowCol = this.editor.selection.getCursor()
      if ((this.readOnlyLines.includes(rowCol.row + 1))) {
        event.preventDefault()
        event.stopPropagation()
      }
    }
  }
}
</script>

<style lang="css">
.exercise-editor-ace-editor {
  position: relative;
}

.readonly-highlight {
  background-color: green;
  opacity: 0.2;
  position: absolute;
}
.hide-in-template-highlight {
  background-color: #2196f3;
  opacity: 0.2;
  position: absolute;
}
.ace_gutter-cell.readonly-breakpoint {
  background-color: green;
  opacity: 0.4;
  color: black;
}
.ace_gutter-cell.hide-in-template-breakpoint {
  background-color: #2196f3;
  opacity: 0.4;
  color: black;
}
.greenMarker {
  position:absolute;
  background:rgba(0,255,0,0.2);
  z-index:20
}
.blueMarker {
  position:absolute;
  background:rgba(0,0,255,0.2);
  z-index:20
}
.greyMarker {
  position:absolute;
  background:rgba(255,255,255,0.5);
  z-index:20
}
</style>
