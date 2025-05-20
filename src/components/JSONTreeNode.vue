<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, computed } from 'vue';
import JSONTreeNode from './JSONTreeNode.vue';

const props = withDefaults(defineProps<{
  nodeKey: string | number;
  nodeValue: any;
  depth?: number;
  baseIndent?: number;
  allowKeyEdit?: boolean;
  nodeClass?: string
  toggleClass?: string
  keyClass?: string
  keyInputClass?: string
  valueClass?: string
  valueInputClass?: string
  childrenClass?: string
  inputClass?: string
}>(), {
  depth: 0,
  baseIndent: 20,
});

const emit = defineEmits<{
  (e: 'update-value', value: any): void;
  (e: 'rename-key', payload: { oldKey: string | number; newKey: string; value: any }): void;
  (e: 'remove-self', key: string | number): void;
}>();

const expanded = ref(false);
const editing = ref(false);
const keyEditMode = ref(false);
const editableValue = ref(props.nodeValue);
const editableKey = ref(String(props.nodeKey));

const displayKey = computed(() => {
  return editableKey.value.trim() === '' ? '<empty>' : editableKey.value;
});

const displayValue = computed(() => {
  if (isObjectOrArray) {
    return expanded.value ? '' : (Array.isArray(props.nodeValue) ? '[Array]' : '{Object}');
  }
  return typeof props.nodeValue === 'string' && props.nodeValue.trim() === '' ? '<empty>' : props.nodeValue;
});

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
  let updatedNode;

  if (Array.isArray(props.nodeValue)) {
    updatedNode = [...props.nodeValue];
    updatedNode[Number(childKey)] = updatedValue;
  } else {
    updatedNode = { ...props.nodeValue, [childKey]: updatedValue };
  }

  emitUpdatedValue(updatedNode);
}
function handleChildKeyRename({ oldKey, newKey, value }: { oldKey: string | number; newKey: string; value: any }) {
  if (Array.isArray(props.nodeValue)) {
    const arr = [...props.nodeValue];
    const index = Number(oldKey);
    if (!isNaN(index)) {
      arr.splice(index, 1); // remove old
      arr.splice(Number(newKey), 0, value); // insert new at new index
      emitUpdatedValue(arr);
    }
  } else {
    const updated = { ...props.nodeValue };
    delete updated[oldKey];
    updated[newKey] = value;
    emitUpdatedValue(updated);
  }
}

// Add
function addChild() {
  if (Array.isArray(editableValue.value)) {
    editableValue.value.push('value');
  } else if (typeof editableValue.value === 'object' && editableValue.value !== null) {
    editableValue.value['newKey_' + Date.now()] = 'value';
  }
  emit('update-value', editableValue.value);
}
// Remove
function removeSelf() {
  emit('remove-self', props.nodeKey);
}
function removeChild(key: string | number) {
  if (Array.isArray(props.nodeValue)) {
    const updated = [...props.nodeValue];
    updated.splice(Number(key), 1);
    emitUpdatedValue(updated);
  } else {
    const updated = { ...props.nodeValue };
    delete updated[key];
    emitUpdatedValue(updated);
  }
}

function emitUpdatedValue(value: any) {
  emit('update-value', value);
}

// Click outside input save and hide input
const inputRef = ref<HTMLInputElement | null>(null);
const keyInputRef = ref<HTMLElement | null>(null)
function handleClickOutside(event: MouseEvent) {
  const target = event.target as Node
  if (editing.value && inputRef.value && !inputRef.value.contains(target)) {
    saveEdit();
  }
  if (keyEditMode.value && keyInputRef.value && !keyInputRef.value.contains(target)) {
    saveKeyEdit()
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
    <div :class="['v3jte-node', nodeClass]" :style="{ paddingLeft: depth * baseIndent + 'px' }">
            <span
                :class="['v3jte-toggle', toggleClass]"
                v-if="isObjectOrArray"
                @click="toggle"
            >
                <slot name="toggle-icon" :expanded="expanded">
                    {{ expanded ? '- ' : '+ ' }}
                </slot>
            </span>
      <span v-if="!keyEditMode" :class="['v3jte-key', keyClass]" :title="allowKeyEdit ? 'Double-click to edit key' : ''"
            @dblclick="allowKeyEdit && (keyEditMode = true)">
              {{ displayKey }}:
            </span>
      <input
          ref="keyInputRef"
          v-if="allowKeyEdit"
          v-show="keyEditMode"
          v-model="editableKey"
          @keyup.enter="saveKeyEdit"
          @keyup.esc="cancelKeyEdit"
          :class="['v3jte-input v3jte-key-input', inputClass, keyInputClass]"
      />
      <span :class="['v3jte-value', valueClass]" :title="!isObjectOrArray ? 'Double-click to edit value' : 'Double-click to expand'"
            @dblclick="isObjectOrArray ? toggle() : (editing = true)"
            v-if="!editing">
                {{ displayValue }}
            </span>
      <input
          ref="inputRef"
          v-if="!isObjectOrArray"
          v-show="editing"
          v-model="editableValue"
          @keyup.enter="saveEdit"
          @keyup.esc="cancelEdit"
          :class="['v3jte-input v3jte-value-input', inputClass, valueInputClass]"
      />
      <span title="Remove">
                <slot name="remove-node" :remove="removeSelf">
                    <button @click="removeSelf" style="margin-left: 10px;">x</button>
                </slot>
            </span>
    </div>

    <ul v-show="expanded" v-if="isObjectOrArray" :class="['v3jte-children', childrenClass]">
      <JSONTreeNode
          v-for="(value, key) in nodeValue"
          :key="key"
          :nodeKey="key"
          :nodeValue="value"
          :depth="depth + 1"
          :base-indent="baseIndent"
          :allow-key-edit="allowKeyEdit"
          :node-class="nodeClass"
          :toggle-class="toggleClass"
          :key-class="keyClass"
          :key-input-class="keyInputClass"
          :value-class="valueClass"
          :value-input-class="valueInputClass"
          :children-class="childrenClass"
          :input-class="inputClass"
          @update-value="(val) => updateChildNode(val, key)"
          @rename-key="handleChildKeyRename"
          @remove-self="removeChild"
      >
        <template #toggle-icon="{ expanded }">
          <slot name="toggle-icon" :expanded="expanded" />
        </template>
        <template #add-child="{ addChild }">
          <slot name="add-child" :addChild="addChild" />
        </template>
        <template #remove-node="{ remove }">
          <slot name="remove-node" :remove="remove" />
        </template>
      </JSONTreeNode>
    </ul>

    <span v-if="isObjectOrArray && expanded" :style="{ paddingLeft: (depth + 1) * baseIndent + 'px' }" title="Add child">
            <slot name="add-child" :addChild="addChild">
                <button @click="addChild">Add</button>
            </slot>
        </span>
  </li>
</template>