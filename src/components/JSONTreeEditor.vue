<script setup lang="ts">
import { ref, watch } from 'vue';
import JSONTreeNode from "./JSONTreeNode.vue";

const props = withDefaults(defineProps<{
  modelValue: Record<string, any> | null;
  baseIndent?: number;
  readOnly?: boolean;
  allowKeyEdit?: boolean;
  allowRowAdding?: boolean;
  allowChildAdding?: boolean;
  allowRemoving?: boolean;
  allowTypeChanging?: boolean;
  containerClass?: string;
  rootClass?: string;
  nodeClass?: string;
  toggleClass?: string;
  keyClass?: string;
  keyInputClass?: string;
  valueClass?: string;
  valueInputClass?: string;
  childrenClass?: string;
  inputClass?: string;
  addRowClass?: string;
  addChildClass?: string;
  removeClass?: string;
  typeSwitchClass?: string;
}>(), {
  baseIndent: 20,
  readOnly: false,
  allowKeyEdit: false,
  allowRowAdding: false,
  allowChildAdding: false,
  allowRemoving: false,
  allowTypeChanging: false,
});

const emit = defineEmits<{
  (e: 'update:modelValue', value: Record<string, any>): void;
  (e: 'invalid-key', payload: { reason: 'non-numeric-in-array' | 'duplicate'; attemptedKey: string; originalKey: string }): void;
}>();

const treeData = ref(props.modelValue || {});

// Sync the internal treeData with the external modelValue
watch(() => props.modelValue, (newValue) => {
  treeData.value = newValue || {};
});

// Emit updated tree data
function updateNode(value: any) {
  emit('update:modelValue', value);
}

// Add
function addRow(payload: { key: string | number }) {
  if (typeof treeData.value === 'object' && !Array.isArray(treeData.value)) {
    const newKey = 'newKey_' + Date.now();
    treeData.value[newKey] = '';
    emit('update:modelValue', { ...treeData.value });
  }
}
// Remove
function removeRow(key: string | number) {
  if (Array.isArray(treeData.value)) {
    treeData.value.splice(Number(key), 1);
  } else {
    const updated = { ...treeData.value };
    delete updated[key];
    treeData.value = updated;
  }
  emit('update:modelValue', { ...treeData.value });
}
</script>

<template>
  <div :class="['v3jte-container', containerClass]">
    <ul :class="['v3jte-root', rootClass]">
      <JSONTreeNode
          v-for="(value, key) in treeData"
          :key="key"
          :nodeKey="key"
          :nodeValue="value"
          :siblingKeys="Object.keys(treeData)"
          :baseIndent="baseIndent"
          :read-only="readOnly"
          :allowKeyEdit="allowKeyEdit"
          :allow-child-adding="allowChildAdding"
          :allow-removing="allowRemoving"
          :allow-type-changing="allowTypeChanging"
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
          @update-value="(val) => updateNode({ ...treeData, [key]: val })"
          @remove-self="removeRow"
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

      <div v-if="!readOnly && allowRowAdding" :class="['v3jte-add-row', addRowClass]" title="Add a new record">
        <slot name="add-row" :addRow="addRow">
          <button @click="addRow">+ Add new</button>
        </slot>
      </div>
    </ul>
  </div>
</template>
