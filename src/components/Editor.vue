<template>
  <div
    class="editor"
    contenteditable="true"
    ref="editor"
    v-text="content"
    spellcheck="false"
    @input="onInput"
    @keydown.left="onLeftKey"
    @keydown.right="onRightKey"
    @click="onClick"
    @paste="onPaste"
  />
</template>

<script>

export default {
  props: {
    value: {
      type: String,
      required: true
    },
    mergeTagClass: {
      type: String,
      default: 'merge-tag'
    },
    mergeTags: {
      type: Object,
      default () {
        return {
          ':name': 'Client Name',
          ':company': 'Company',
          ':age': 'Age'
        }
      }
    }
  },
  data () {
    return {
      content: this.value
    }
  },
  methods: {
    onInput () {
      this.checkTags()
      this.removeStyles()
      this.insertMergeTags()
      this.$emit('input', this.cleanContent(this.$el.innerHTML))
    },
    onLeftKey (e) {
      if (document.getSelection().anchorOffset > 1) return

      const element = document.getSelection().anchorNode
      const previousElement = element.previousSibling

      if (previousElement && previousElement.classList && previousElement.classList.contains(this.mergeTagClass)) {
        this.placeCarretBeforeElement(previousElement)
        e.preventDefault()
      }
    },
    onRightKey (e) {
      const selection = document.getSelection()
      if (selection.anchorNode.nodeType === 3) {
        if (selection.anchorOffset < selection.anchorNode.nodeValue.length) return
      }

      const element = document.getSelection().anchorNode
      const currentElementIsMergeTag = element.classList && element.classList.contains(this.mergeTagClass)
      const nextElement = currentElementIsMergeTag ? element : element.nextSibling

      if (nextElement && nextElement.classList && nextElement.classList.contains(this.mergeTagClass)) {
        this.placeCarretAfterElement(nextElement)
        e.preventDefault()
      }
    },
    onClick (e) {
      const element = this.getElementAtCarret()
      const currentElementIsMergeTag = element.classList && element.classList.contains(this.mergeTagClass)
      if (currentElementIsMergeTag) {
        this.placeCarretAfterElement(element)
      }
    },
    onPaste (e) {
      e.preventDefault()
      this.pasteElementAtCaret(document.createTextNode(e.clipboardData.getData('text/plain')), false)
      this.onInput()
    },
    insertMergeTags () {
      const selection = window.getSelection()
      const nodes = this.$el.childNodes
      for (var i = 0; i < nodes.length; i++) {
        const node = nodes[i]
        if (node.nodeType === 3) {
          Object.keys(this.mergeTags).forEach(needle => {
            const startIndex = node.nodeValue.indexOf(needle)
            if (startIndex !== -1) {
              const range = document.createRange()
              range.setStart(node, startIndex)
              range.setEnd(node, startIndex + needle.length)
              selection.removeAllRanges()
              selection.addRange(range)

              const element = this.createTag(needle, this.mergeTags[needle])
              this.insertElementAtRange(element, range, false)
            }
          })
        }
      }
    },
    checkTags () {
      const mergeTags = this.$el.getElementsByClassName(this.mergeTagClass)
      for (var i = 0; i < mergeTags.length; i++) {
        if (mergeTags[i].innerText.length < mergeTags[i].dataset.text.length) {
          this.removeTag(mergeTags[i])
        } else if (mergeTags[i].innerText.toLowerCase() !== mergeTags[i].dataset.text.toLowerCase()) {
          mergeTags[i].innerText = mergeTags[i].dataset.text
          this.placeCarretBeforeElement(mergeTags[i])
        }
      }
    },
    removeStyles () {
      const nodes = this.$el.childNodes
      for (var i = 0; i < nodes.length; i++) {
        if (nodes[i].style) nodes[i].style = null
      }
    },
    removeTag (element) {
      const range = document.createRange()
      range.selectNode(element)
      range.deleteContents()
    },
    addMergeTag (mergeTag) {
      this.pasteElementAtCaret(this.createTag(mergeTag, this.mergeTags[mergeTag]))
      this.onInput()
    },
    createTag (tag, text) {
      const el = document.createElement('span')
      el.innerText = text
      el.dataset.text = text
      el.dataset.mergeTag = tag
      el.contenteditable = false
      el.className = this.mergeTagClass

      return el
    },
    pasteElementAtCaret (element, wrapWithSpaces = true) {
      this.$el.focus()

      const selection = window.getSelection()

      if (selection.getRangeAt && selection.rangeCount) {
        const range = selection.getRangeAt(0)
        this.insertElementAtRange(element, range, wrapWithSpaces)
      }
    },
    insertElementAtRange  (element, range, wrapWithSpaces = true) {
      var documentFragment = document.createDocumentFragment()

      // if (wrapWithSpaces) documentFragment.appendChild(document.createTextNode(' '))
      documentFragment.appendChild(element)
      documentFragment.appendChild(document.createTextNode(wrapWithSpaces ? ' ' : '\u200B'))

      range.deleteContents()
      range.insertNode(documentFragment)

      this.placeCarretAfterElement(element)
    },
    placeCarretAfterElement (element) {
      element = element.nextSibling
      if (!element) {
        element = document.createTextNode('\u200B')
        this.$el.appendChild(element)
      } else {
        if (element.nodeType === 3 && element.nodeValue[0] !== '\u200B') {
          element.nodeValue = '\u200B' + element.nodeValue
        }
      }
      const selection = window.getSelection()
      const range = selection.getRangeAt(0)
      range.setStart(element, 1)
      range.collapse(false)
      selection.removeAllRanges()
      selection.addRange(range)
    },
    placeCarretBeforeElement (element) {
      const selection = window.getSelection()
      const range = selection.getRangeAt(0)
      range.setStart(element, 0)
      range.collapse(true)
      selection.removeAllRanges()
      selection.addRange(range)
    },
    getElementAtCarret () {
      const node = document.getSelection().anchorNode
      return (node.nodeType === 3 ? node.parentNode : node)
    },
    cleanContent (content) {
      const dummy = document.createElement('div')
      dummy.innerHTML = content.replace('<div', '\n<div').replace('<br>', '\n\n')

      const mergeTags = dummy.getElementsByClassName(this.mergeTagClass)
      for (var i = 0; i < mergeTags.length; i++) {
        mergeTags[i].innerText = mergeTags[i].dataset.mergeTag
      }

      return dummy.innerText
    }
  },
  mounted () {
    this.insertMergeTags()
  }
}
</script>
<style lang="scss">
.editor {
  border: 1px dashed #ddd;
  text-align: left;
  min-height: 100px;
  padding: 10px;
  line-height: 1.5rem;
  b {
    font-weight: inherit;
  }
  i {
    font-style: inherit;
  }
}
.merge-tag {
  padding: 1px 3px;
  border: 1px solid #cce;
  background-color: #ddf;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 0.8em;
  text-transform: uppercase;
  box-shadow: 1px 1px 0px 0 rgba(0,0,0,0.05);
  margin: 0 1px;
}
</style>
