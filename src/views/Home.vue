<template>
  <div class="home">
    <h2>JSON Viewer</h2>

    <div
      class="drop"
      :class="{ over: dragging }"
      @dragover.prevent="dragging = true"
      @dragleave.prevent="dragging = false"
      @drop.prevent="onDrop"
      @click="$refs.file.click()"
    >
      JSON 파일을 여기로 드래그하거나 클릭해서 선택
      <input ref="file" type="file" accept=".json,application/json" hidden @change="onPick">
    </div>

    <p v-if="error" class="error">⚠ {{ error }}</p>

    <template v-if="parsed !== undefined">
      <div class="toolbar">
        <span class="name">{{ fileName }}</span>
        <button @click="setAll(true)">모두 펼치기</button>
        <button @click="setAll(false)">모두 접기</button>
        <button @click="copy">{{ copied ? '복사됨 ✓' : '클립보드 복사' }}</button>
        <button @click="saveTxt">TXT 저장</button>
      </div>
      <div class="tree">
        <json-node :value="parsed" :expand-all="expandAll" />
      </div>
    </template>
  </div>
</template>

<script>
import JsonNode from '@/components/JsonNode.vue'

export default {
  name: 'Home',
  components: { JsonNode },
  data () {
    return {
      parsed: undefined,
      fileName: '',
      error: '',
      dragging: false,
      copied: false,
      expandAll: { open: false, tick: 0 }
    }
  },
  computed: {
    text () { return JSON.stringify(this.parsed, null, 2) }
  },
  methods: {
    onDrop (e) {
      this.dragging = false
      this.load(e.dataTransfer.files[0])
    },
    onPick (e) {
      this.load(e.target.files[0])
      e.target.value = ''
    },
    load (file) {
      if (!file) return
      const reader = new FileReader()
      reader.onload = () => {
        try {
          // BOM 제거 후 파싱
          this.parsed = JSON.parse(String(reader.result).replace(/^\uFEFF/, ''))
          this.fileName = file.name
          this.error = ''
          this.setAll(false)
        } catch (err) {
          this.parsed = undefined
          this.error = `JSON 파싱 실패: ${err.message}`
        }
      }
      reader.onerror = () => { this.error = '파일을 읽을 수 없습니다.' }
      reader.readAsText(file)
    },
    setAll (open) {
      this.expandAll = { open, tick: this.expandAll.tick + 1 }
    },
    async copy () {
      try {
        if (navigator.clipboard && window.isSecureContext) {
          await navigator.clipboard.writeText(this.text)
        } else {
          const ta = document.createElement('textarea')
          ta.value = this.text
          document.body.appendChild(ta)
          ta.select()
          document.execCommand('copy')
          document.body.removeChild(ta)
        }
        this.copied = true
        setTimeout(() => { this.copied = false }, 1500)
      } catch (err) {
        this.error = '복사 실패: ' + err.message
      }
    },
    saveTxt () {
      const blob = new Blob([this.text], { type: 'text/plain;charset=utf-8' })
      const a = document.createElement('a')
      a.href = URL.createObjectURL(blob)
      a.download = (this.fileName.replace(/\.json$/i, '') || 'output') + '.txt'
      a.click()
      URL.revokeObjectURL(a.href)
    }
  }
}
</script>

<style scoped>
.home { max-width: 900px; margin: 0 auto; padding: 0 16px 40px; text-align: left; }
h2 { text-align: center; }
.drop { border: 2px dashed #aaa; border-radius: 8px; padding: 32px; text-align: center; color: #666; cursor: pointer; }
.drop.over { border-color: #42b983; background: #eefaf3; }
.error { color: #c62828; }
.toolbar { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; margin: 16px 0 8px; }
.name { font-weight: bold; margin-right: auto; }
button { padding: 6px 12px; border: 1px solid #ccc; border-radius: 4px; background: #fff; cursor: pointer; }
button:hover { background: #f0f4f8; }
.tree { border: 1px solid #ddd; border-radius: 6px; padding: 12px; overflow: auto; max-height: 70vh; }
</style>
