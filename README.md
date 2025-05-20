# vue3-json-tree-editor

A customizable and lightweight JSON tree editor component for Vue 3.  

---

## ✨ Key Features

- Edit JSON keys and values
- Expand/collapse object and array nodes
- Visual indentation based on nesting depth

## 📦 Installation

```bash
npm install vue3-json-tree-editor
```

## 🚀 Usage

### Basic

```vue
<script setup>
import JSONTreeEditor from 'vue3-json-tree-editor'

const JSONTreeData = ref({
  name: 'John',
  age: 30,
  skills: {
    frontend: 'Vue',
    backend: 'Node.js'
  },
})
</script>

<template>
  <JSONTreeEditor v-model="JSONTreeData" />
</template>
```

### Full

```vue
<JSONTreeEditor
  v-model="JSONTreeData"
  :allow-key-edit="true"
  :base-indent="20"
>
  <template #toggle-icon="{ expanded }">
    <span>{{ expanded ? '▼ ' : '▶ ' }}</span>
  </template>
</JSONTreeEditor>
```

## 🧩 CSS Class Reference

Use these classes to customize the appearance of the component:

- `v3jte-root` – Root wrapper of the entire JSON editor
- `v3jte-container` – Container for each JSON node row
- `v3jte-node` – Main wrapper for a single key-value pair or branch
- `v3jte-toggle` – Expand/collapse button for objects and arrays
- `v3jte-key` – Key label display
- `v3jte-key-input` – Input field for editing keys
- `v3jte-value` – Value label display
- `v3jte-value-input` – Input field for editing values
- `v3jte-children` – Wrapper for nested child nodes
- `v3jte-input` – Shared base class applied to all editable input fields
