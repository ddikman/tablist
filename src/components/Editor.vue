<script setup lang="ts">
import { computed, ref, watch } from 'vue'

const EXAMPLE = `This is
  An example of
  How the nesting
  Works
Each parent
  Will have the children in a sub-column
`

const KEY = 'editor_text'
const TAB = '  '

const text = ref(localStorage.getItem(KEY) || EXAMPLE)

watch(text, () => {
  if (text.value.length === 0) {
    localStorage.removeItem(KEY)
  } else {
    localStorage.setItem(KEY, text.value)
  }
})

const formatTags = (txt: string) => {
  return '<span>' + txt.replace(/#([^\s]+)/g, '<span class="confirm" style="color: #ff0000; font-weight: bold;">#$1</span>')
}

const structured = computed(() => {
  const rows = text.value.split('\n')
  const nodes = new Array<{ children: string[], content: string }>()
  for (const row of rows) {
    if (row.startsWith(TAB)) {
      const parent = nodes[nodes.length - 1]
      if (row.trim().toLocaleLowerCase().startsWith('confirm')) {
        // add an empty row
        parent.children.push('')
      }
      parent.children.push(formatTags(row.trim()))
    } else {
      nodes.push({ content: formatTags(row), children: [] })
    }
  }
  return nodes
})

function addTab(event: any) {
  event.preventDefault()
  let startText = text.value.slice(0, event.target.selectionStart)
  let endText = text.value.slice(event.target.selectionStart)
  text.value = `${startText}${TAB}${endText}`
  event.target.selectionEnd = event.target.selectionStart + 1
}

function copyText() {
  navigator.clipboard.writeText(text.value)
}

const tableRef = ref<HTMLElement>()
async function copyTable() {
  if (!tableRef.value) { 
    return 
  }
  
  const selection: any = window.getSelection();

  const range = document.createRange();
  range.selectNodeContents( tableRef.value );
  selection.removeAllRanges();
  selection.addRange( range );
  document.execCommand( 'copy' );
  window.getSelection()?.removeAllRanges();
}

</script>

<template>
  <div class="content">
    <div class="input">
      <span class="copy" @click="copyText">copy</span>
      <textarea @keydown.tab.prevent="addTab" v-model="text"></textarea>
    </div>
    <div class="table">
      <span class="copy" @click="copyTable">copy</span>
      <table ref="tableRef">
        <tr v-for="entry in structured">
          <td v-html="entry.content"></td>
          <td v-html="entry.children.join('<br/>')"></td>
        </tr>
      </table>
    </div>
  </div>
</template>

<style>
.content {
  display: flex;
  flex-direction: row;
  height: 100vh;
  width: 90vw;
  gap: 32px;
}

.confirm {
  color: red;
  font-weight: bold;
}

.tag {
  color: green;
  font-weight: bold;
}
.content > * {
  width: 100%;
  height: 100%;
}

.input {
  display: flex;
  flex-direction: column;  
}

.input textarea {
  flex: 1;
}

td {
  text-align: left;
  vertical-align: top;
}

tr:nth-child(odd) {
  background-color: lightgray;
}

.copy:hover {
  cursor: pointer;
  color: blue;
}

</style>
