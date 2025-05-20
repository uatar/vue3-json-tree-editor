<script setup lang="ts">
import { ref, watch } from 'vue';
import JSONTreeNode from "./JSONTreeNode.vue";

const props = withDefaults(defineProps<{
  modelValue: Record<string, any> | null;
  baseIndent?: number;
  allowKeyEdit?: boolean;
  allowRowAdding?: boolean;
  allowChildAdding?: boolean;
  allowRemoving?: boolean;
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
}>(), {
  baseIndent: 20,
  allowKeyEdit: false,
  allowRowAdding: false,
  allowChildAdding: false,
  allowRemoving: false,
});

const emit = defineEmits<{
  (e: 'update:modelValue', value: Record<string, any>): void;
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
          :baseIndent="baseIndent"
          :allowKeyEdit="allowKeyEdit"
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
          @update-value="(val) => updateNode({ ...treeData, [key]: val })"
          @remove-self="removeRow"
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

      <div v-if="allowRowAdding" :class="['v3jte-add-row', addRowClass]" title="Add a new record">
        <slot name="add-row" :addRow="addRow">
          <button @click="addRow">+ Add new</button>
        </slot>
      </div>
    </ul>
  </div>
</template>
