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
  :container-class="'p-3'"
  :root-class="'p-3 shadow'"
  :node-class="'hover:shadow-sm'"
  :toggle-class="'cursor-pointer'"
  :key-class="''"
  :value-class="''"
  :input-class="'border-neutral-300 dark:border-neutral-700 dark:bg-neutral-900 dark:text-neutral-300 focus:border-primary-500 dark:focus:border-primary-600 focus:ring-primary-500 dark:focus:ring-primary-600 shadow-sm'"
  :key-input-class="'mr-3'"
  :value-input-class="''"
  :children-class="''"
>
  <template #toggle-icon="{ expanded }">
    <span>{{ expanded ? '⯆ ' : '⯈ ' }}</span>
  </template>
</JSONTreeEditor>
```

## 🧩 CSS Class Reference

Use these classes to customize the appearance of the component:

- `v3jte-container` – Container of the entire JSON editor
- `v3jte-root` – Root wrapper of the entire JSON editor
- `v3jte-node` – Main wrapper for a single key-value pair or branch
- `v3jte-toggle` – Expand/collapse button for objects and arrays
- `v3jte-key` – Key label display
- `v3jte-key-input` – Input field for editing keys
- `v3jte-value` – Value label display
- `v3jte-value-input` – Input field for editing values
- `v3jte-children` – Wrapper for nested child nodes
- `v3jte-input` – Shared base class applied to all editable input fields
