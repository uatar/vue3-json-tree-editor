<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref } from 'vue';
import JSONTreeNode from './JSONTreeNode.vue';

const props = withDefaults(defineProps<{
  nodeKey: string | number;
  nodeValue: any;
  depth?: number;
  baseIndent?: number;
  allowKeyEdit?: boolean;
}>(), {
  depth: 0,
  baseIndent: 20,
});

const emit = defineEmits<{
  (e: 'update-value', value: any): void;
  (e: 'rename-key', payload: { oldKey: string | number; newKey: string; value: any }): void;
}>();

const expanded = ref(false);
const editing = ref(false);
const keyEditMode = ref(false);
const editableValue = ref(props.nodeValue);
const editableKey = ref(String(props.nodeKey));

const isObjectOrArray = typeof props.nodeValue === 'object' && props.nodeValue !== null;

function toggle() {
  expanded.value = !expanded.value;
}
function saveEdit() {
  editing.value = false;
  emitUpdatedValue(editableValue.value);
}
function cancelEdit() {
  editing.value = false;
  editableValue.value = props.nodeValue;
}
function saveKeyEdit() {
  keyEditMode.value = false;

  const newKey = editableKey.value.trim();
  const oldKey = props.nodeKey;

  if (!newKey || newKey === oldKey) return;

  emit('rename-key', {
    oldKey,
    newKey,
    value: editableValue.value ?? props.nodeValue, // ensure value goes with it
  });
}
function cancelKeyEdit() {
  keyEditMode.value = false;
  editableKey.value = String(props.nodeKey);
}

function updateChildNode(updatedValue: any, childKey: string | number) {
  const updatedNode = { ...props.nodeValue, [childKey]: updatedValue };
  emitUpdatedValue(updatedNode);
}
function handleChildKeyRename({ oldKey, newKey, value }: { oldKey: string | number; newKey: string; value: any }) {
  if (typeof props.nodeValue !== 'object' || props.nodeValue === null) return;

  const entries = Object.entries(props.nodeValue);
  const updatedEntries = entries.map(([key, val]) =>
      key === oldKey ? [newKey, value] : [key, val]
  );

  const updatedObject = Object.fromEntries(updatedEntries);
  emitUpdatedValue(updatedObject);
}

function emitUpdatedValue(value: any) {
  emit('update-value', value);
}

// Click outside input save and hide input
const inputRef = ref<HTMLInputElement | null>(null);
function handleClickOutside(event: MouseEvent) {
  if (editing.value && inputRef.value && !inputRef.value.contains(event.target as Node)) {
    saveEdit();
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside);
});
onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside);
});
</script>

<template>
  <li>
    <div class="tree-node" :style="{ paddingLeft: depth * baseIndent + 'px' }">
            <span
                class="tree-node-toggle"
                v-if="isObjectOrArray"
                @click="toggle"
            >
                {{ expanded ? '- ' : '+ ' }}
            </span>
      <span v-if="!keyEditMode" class="tree-node-key" @dblclick="allowKeyEdit && (keyEditMode = true)">
              {{ editableKey }}:
            </span>
      <input
          v-if="keyEditMode"
          v-model="editableKey"
          @keyup.enter="saveKeyEdit"
          @keyup.esc="cancelKeyEdit"
          @blur="saveKeyEdit"
      />
      <span class="tree-node-value"
            @dblclick="isObjectOrArray ? toggle() : (editing = true)"
            v-if="!editing">
                {{ isObjectOrArray ? (expanded ? '' : (Array.isArray(nodeValue) ? '[Array]' : '{Object}')) : nodeValue }}
            </span>
      <input
          v-if="!isObjectOrArray"
          ref="inputRef"
          v-model="editableValue"
          v-show="editing"
          @keyup.enter="saveEdit"
          @keyup.esc="cancelEdit"
      />
    </div>

    <ul v-show="expanded" v-if="isObjectOrArray">
      <JSONTreeNode
          v-for="(value, key) in nodeValue"
          :key="key"
          :nodeKey="key"
          :nodeValue="value"
          :depth="depth + 1"
          :base-indent="baseIndent"
          :allow-key-edit="allowKeyEdit"
          @update-value="(val) => updateChildNode(val, key)"
          @rename-key="handleChildKeyRename"
      />
    </ul>
  </li>
</template>
