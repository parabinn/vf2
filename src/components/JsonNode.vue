<template>
  <div class="node">
    <div class="row" :class="{ clickable: expandable }" @click="toggle">
      <span class="arrow">{{ expandable ? (open ? '▼' : '▶') : '' }}</span>
      <span v-if="label !== null" class="key">{{ label }}</span>
      <span v-if="label !== null" class="colon">:</span>
      <template v-if="expandable">
        <span class="brace">{{ isArray ? '[' : '{' }}</span>
        <span v-if="!open" class="summary">
          {{ count }} {{ isArray ? 'items' : 'keys' }} {{ isArray ? ']' : '}' }}
        </span>
      </template>
      <span v-else :class="['val', typeClass]">{{ display }}</span>
    </div>
    <div v-if="expandable && open" class="children">
      <json-node
        v-for="child in children"
        :key="child.key"
        :label="child.key"
        :value="child.value"
        :depth="depth + 1"
        :expand-all="expandAll"
      />
      <div class="row"><span class="arrow"></span><span class="brace">{{ isArray ? ']' : '}' }}</span></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'JsonNode',
  props: {
    label: { type: [String, Number], default: null },
    value: { default: null },
    depth: { type: Number, default: 0 },
    // 값이 바뀔 때마다(펼치기/접기 버튼) 전체 상태를 덮어쓰기 위한 {open, tick}
    expandAll: { type: Object, default: () => ({ open: true, tick: 0 }) }
  },
  data () {
    return { open: this.expandAll.open || this.depth < 2 }
  },
  computed: {
    isArray () { return Array.isArray(this.value) },
    expandable () { return this.value !== null && typeof this.value === 'object' },
    children () {
      if (!this.expandable) return []
      const keys = this.isArray ? this.value.map((_, i) => i) : Object.keys(this.value)
      return keys.map(k => ({ key: k, value: this.value[k] }))
    },
    count () { return this.children.length },
    typeClass () { return this.value === null ? 'null' : typeof this.value },
    display () {
      return typeof this.value === 'string' ? JSON.stringify(this.value) : String(this.value)
    }
  },
  watch: {
    'expandAll.tick' () { this.open = this.expandAll.open }
  },
  methods: {
    toggle () { if (this.expandable) this.open = !this.open }
  }
}
</script>

<style scoped>
.node { font-family: Consolas, Menlo, monospace; font-size: 14px; line-height: 1.6; }
.children { margin-left: 18px; border-left: 1px dotted #ccc; padding-left: 6px; }
.row { white-space: pre-wrap; word-break: break-all; }
.clickable { cursor: pointer; }
.clickable:hover { background: #f0f4f8; }
.arrow { display: inline-block; width: 14px; font-size: 10px; color: #888; }
.key { color: #9c27b0; }
.colon, .brace { color: #555; margin-right: 4px; }
.summary { color: #999; font-style: italic; }
.string { color: #2e7d32; }
.number { color: #1565c0; }
.boolean { color: #e65100; }
.null { color: #888; }
</style>
