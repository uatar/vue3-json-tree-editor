# vue3-json-tree-editor

A lightweight and customizable JSON tree editor component for Vue 3.

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
    :read-only="false"
    :allow-key-edit="true"
    :allow-row-adding="true"
    :allow-child-adding="true"
    :allow-removing="true"
    :base-indent="20"
    :container-class="'p-3'"
    :root-class="'p-3 shadow'"
    :node-class="'hover:shadow-sm'"
    :toggle-class="'cursor-pointer'"
    :key-class="''"
    :value-class="''"
    :input-class="'border-neutral-300 dark:border-neutral-700 dark:bg-neutral-900 dark:text-neutral-300 shadow-sm'"
    :key-input-class="'mr-3'"
    :value-input-class="''"
    :children-class="''"
    :add-row-class="'text-green-500 hover:text-green-300'"
    :add-child-class="'text-green-500 hover:text-green-300'"
    :remove-class="'text-red-500 hover:text-red-300'"
    :type-switch-class="'text-cyan-500 hover:text-cyan-300'"
    @invalid-key="(payload) => showError(payload.reason)"
>
    <template #toggle-icon="{ expanded }">
      <span>{{ expanded ? '⯆ ' : '⯈ ' }}</span>
    </template>
    <template #add-row="{ addRow }">
      <button @click="addRow">Add Item</button>
    </template>
    <template #add-child="{ addChild }">
      <button @click="addChild">Add Child</button>
    </template>
    <template #remove-node="{ remove }">
      <button @click="remove" style="margin-left: 10px;">x</button>
    </template>
    <template #type-switch="{ toggle }">
      <button @click="toggle" style="margin-left: 10px;">⇄</button>
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
- `v3jte-add-row` – Wrapper of the button for adding a new top-level key/value
- `v3jte-add-child` – Wrapper of the button for adding a child inside objects/arrays
- `v3jte-remove` – Wrapper of the remove button for deleting a node
- `v3jte-type-switch` – Wrapper of the button for changing the type of values

## ⚠️ Events: Error Handling

These events are emitted when errors occur. You can use them to show user-facing notifications, tooltips, or log invalid input.

### `invalid-key`

Emitted when a key renaming attempt is rejected.

**Payload:**
```ts
{
  reason: string;
  attemptedKey: string;
  originalKey: string;
}
```

Possible reason values:

- 'non-numeric-in-array' – A non-numeric key was entered in an array context.
- 'duplicate' – A key with the same name already exists at the current level.

