<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, computed, watch } from 'vue';
import JSONTreeNode from './JSONTreeNode.vue';

const props = withDefaults(defineProps<{
  nodeKey: string | number;
  nodeValue: any;
  baseIndent?: number;
  readOnly?: boolean;
  allowKeyEdit?: boolean;
  allowChildAdding?: boolean;
  allowRemoving?: boolean;
  nodeClass?: string;
  toggleClass?: string;
  keyClass?: string;
  keyInputClass?: string;
  valueClass?: string;
  valueInputClass?: string;
  childrenClass?: string;
  inputClass?: string;
  addChildClass?: string;
  removeClass?: string;
  typeSwitchClass?: string;

  depth?: number;
  isParentArray?: boolean;
  siblingKeys?: string[];
}>(), {
  depth: 0,
  baseIndent: 20,
  readOnly: false,
  allowKeyEdit: false,
  allowRowAdding: false,
  allowChildAdding: false,
  allowRemoving: false,
  isParentArray: false,
});

const emit = defineEmits<{
  (e: 'update-value', value: any): void;
  (e: 'rename-key', payload: { oldKey: string | number; newKey: string; value: any }): void;
  (e: 'remove-self', key: string | number): void;
  (e: 'invalid-key', payload: { reason: 'non-numeric-in-array' | 'duplicate'; attemptedKey: string; originalKey: string }): void;
}>();

const expanded = ref(false);
const editing = ref(false);
const keyEditMode = ref(false);
const showTypeSelect = ref(false);
const hovering = ref(false);
const editableValue = ref(props.nodeValue);
const editableKey = ref(String(props.nodeKey));

const displayKey = computed(() => {
  return editableKey.value.trim() === '' ? '<empty>' : editableKey.value;
});
const displayValue = computed(() => {
  if (isObjectOrArray) {
    return expanded.value ? '' : (Array.isArray(props.nodeValue) ? '[Array]' : '{Object}');
  }
  if (typeof props.nodeValue === 'string') {
    const trimmed = props.nodeValue.trim();
    return trimmed === '' ? '<empty>' : `'${props.nodeValue}'`;
  }

  return props.nodeValue;
});

watch(() => props.nodeKey, (newKey) => {
  editableKey.value = String(newKey);
});

const isObjectOrArray = typeof props.nodeValue === 'object' && props.nodeValue !== null;

function toggle() {
  expanded.value = !expanded.value;
}
function saveEdit() {
  editing.value = false;
  let newVal = editableValue.value;
  // number validation
  if (typeof props.nodeValue === 'number' && !isNaN(Number(newVal))) {
    newVal = Number(newVal);
  }
  // bool validation
  else if (typeof props.nodeValue === 'boolean') {
    const lower = String(newVal).toLowerCase();
    if (lower === 'true') newVal = true;
    else if (lower === 'false') newVal = false;
    else {
      editableValue.value = props.nodeValue;
      newVal = props.nodeValue;
    }
  }
  emitUpdatedValue(newVal);
}
function cancelEdit() {
  editing.value = false;
  editableValue.value = props.nodeValue;
}
function saveKeyEdit() {
  keyEditMode.value = false;

  const newKey = editableKey.value.trim();
  const oldKey = props.nodeKey;

  if (props.isParentArray && !/^\d+$/.test(newKey)) {
    editableKey.value = String(oldKey); // revert
    emit('invalid-key', {
      reason: 'non-numeric-in-array',
      attemptedKey: newKey,
      originalKey: String(oldKey),
    });
    return;
  }

  if (
      props.siblingKeys &&
      props.siblingKeys.includes(newKey) &&
      newKey !== String(oldKey)
  ) {
    editableKey.value = String(oldKey); // revert
    emit('invalid-key', {
      reason: 'duplicate',
      attemptedKey: newKey,
      originalKey: String(oldKey),
    });
    return;
  }

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
    editableValue.value.push('');
  } else if (typeof editableValue.value === 'object' && editableValue.value !== null) {
    editableValue.value['newKey_' + Date.now()] = '';
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
// Type Change
const supportedTypes = ['string', 'number', 'boolean', 'array', 'object'];
function changeType(newType: string) {
  showTypeSelect.value = false;
  const oldVal = editableValue.value;
  let newVal: any;

  switch (newType) {
    case 'string':
      newVal = typeof oldVal === 'string' ? oldVal : String(oldVal);
      break;
    case 'number':
      if (typeof oldVal === 'number') newVal = oldVal;
      else {
        const parsed = Number(oldVal);
        newVal = isNaN(parsed) ? 0 : parsed;
      }
      break;
    case 'boolean':
      if (typeof oldVal === 'boolean') newVal = oldVal;
      else {
        const str = String(oldVal).toLowerCase();
        if (str === 'true') newVal = true;
        else if (str === 'false') newVal = false;
        else newVal = false;
      }
      break;
    case 'array':
      newVal = Array.isArray(oldVal) ? oldVal : [oldVal];
      break;
    case 'object':
      newVal = typeof oldVal === 'object' && oldVal !== null && !Array.isArray(oldVal)
          ? oldVal
          : { value: oldVal };
      break;
  }

  editableValue.value = newVal;
  emitUpdatedValue(newVal);
}

function emitUpdatedValue(value: any) {
  emit('update-value', value);
}

// Click outside input save and hide input
const inputRef = ref<HTMLInputElement | null>(null);
const keyInputRef = ref<HTMLElement | null>(null)
const typeInputRef = ref<HTMLElement | null>(null);
const typeToggleBtnRef = ref<HTMLElement | null>(null);
function handleClickOutside(event: MouseEvent) {
  const target = event.target as Node
  if (editing.value && inputRef.value && !inputRef.value.contains(target)) {
    saveEdit();
  }
  if (keyEditMode.value && keyInputRef.value && !keyInputRef.value.contains(target)) {
    saveKeyEdit();
  }
  if (
      showTypeSelect.value &&
      typeInputRef.value &&
      !typeInputRef.value.contains(target) &&
      (!typeToggleBtnRef.value || !typeToggleBtnRef.value.contains(target))
  ) {
    showTypeSelect.value = false;
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
    <div :class="['v3jte-node', nodeClass]"
         :style="{ paddingLeft: depth * baseIndent + 'px' }"
         @mouseenter="hovering = true"
         @mouseleave="hovering = false">
            <span
                :class="['v3jte-toggle', toggleClass]"
                v-if="isObjectOrArray"
                @click="toggle"
            >
                <slot name="toggle-icon" :expanded="expanded">
                    {{ expanded ? '- ' : '+ ' }}
                </slot>
            </span>
      <span v-if="!keyEditMode && !props.isParentArray"
            :class="['v3jte-key', keyClass]"
            :title="!readOnly && allowKeyEdit ? 'Double-click to edit key' : ''"
            @dblclick="!readOnly && allowKeyEdit && (keyEditMode = true)">
              {{ displayKey }}:
            </span>
      <input
          ref="keyInputRef"
          v-if="!readOnly && allowKeyEdit && !props.isParentArray"
          v-show="keyEditMode"
          v-model="editableKey"
          @keyup.enter="saveKeyEdit"
          @keyup.esc="cancelKeyEdit"
          :class="['v3jte-input v3jte-key-input', inputClass, keyInputClass]"
      />
      <span
          :class="['v3jte-value', valueClass]"
          :title="!isObjectOrArray ? (!readOnly ? 'Double-click to edit value' : '') : 'Double-click to expand'"
          @dblclick="isObjectOrArray ? toggle() : (editing = !readOnly)"
          v-if="!editing && !showTypeSelect"
      >
                {{ displayValue }}
            </span>
      <select
          ref="typeInputRef"
          v-if="!readOnly"
          v-show="showTypeSelect"
          @change="changeType($event.target.value)"
          @keyup.esc="showTypeSelect = false"
          :class="['v3jte-input v3jte-value-input', inputClass, valueInputClass]"
      >
        <option v-for="type in supportedTypes" :key="type" :value="type">
          {{ type }}
        </option>
      </select>
      <select
          ref="inputRef"
          v-if="!readOnly && typeof props.nodeValue === 'boolean'"
          v-show="editing"
          v-model="editableValue"
          @change="saveEdit"
          @keyup.esc="cancelEdit"
          :class="['v3jte-input v3jte-value-input', inputClass, valueInputClass]"
      >
        <option :value="true">true</option>
        <option :value="false">false</option>
      </select>
      <input
          ref="inputRef"
          :type="typeof props.nodeValue === 'number' ? 'number' : 'text'"
          v-else-if="!readOnly && !isObjectOrArray"
          v-show="editing"
          v-model="editableValue"
          @keyup.enter="saveEdit"
          @keyup.esc="cancelEdit"
          :class="['v3jte-input v3jte-value-input', inputClass, valueInputClass]"
      />
      <span v-if="!readOnly && hovering" :class="['v3jte-type-switch', typeSwitchClass]" title="Change Type" ref="typeToggleBtnRef">
                <slot name="type-switch" :toggle="() => showTypeSelect = !showTypeSelect">
                    <button @click="showTypeSelect = !showTypeSelect" style="margin-left: 10px;">T</button>
                </slot>
            </span>
      <span v-if="!readOnly && allowRemoving && hovering" :class="['v3jte-remove', removeClass]" title="Remove">
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
          :is-parent-array="Array.isArray(nodeValue)"
          :sibling-keys="Object.keys(nodeValue)"
          :depth="depth + 1"
          :base-indent="baseIndent"
          :read-only="readOnly"
          :allow-key-edit="allowKeyEdit"
          :allow-child-adding="allowChildAdding"
          :allow-removing="allowRemoving"
          :node-class="nodeClass"
          :toggle-class="toggleClass"
          :key-class="keyClass"
          :key-input-class="keyInputClass"
          :value-class="valueClass"
          :value-input-class="valueInputClass"
          :children-class="childrenClass"
          :input-class="inputClass"
          :add-child-class="addChildClass"
          :remove-class="removeClass"
          :type-switch-class="typeSwitchClass"
          @update-value="(val) => updateChildNode(val, key)"
          @rename-key="handleChildKeyRename"
          @remove-self="removeChild"
          @invalid-key="$emit('invalid-key', $event)"
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
        <template #type-switch="{ toggle }">
          <slot name="type-switch" :toggle="toggle"/>
        </template>
      </JSONTreeNode>
    </ul>

    <span v-if="!readOnly && allowChildAdding && isObjectOrArray" v-show="expanded" :style="{ paddingLeft: (depth + 1) * baseIndent + 'px' }"
          :class="['v3jte-add-child', addChildClass]" title="Add a new record">
            <slot name="add-child" :addChild="addChild">
                <button @click="addChild">Add</button>
            </slot>
        </span>
  </li>
</template>
