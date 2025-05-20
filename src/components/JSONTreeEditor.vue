<script setup lang="ts">
import { ref, watch } from 'vue';
import JSONTreeNode from "./JSONTreeNode.vue";

const props = withDefaults(defineProps<{
  modelValue: Record<string, any> | null;
  baseIndent?: number;
  allowKeyEdit?: boolean;
  containerClass?: string
  rootClass?: string
  nodeClass?: string
  toggleClass?: string
  keyClass?: string
  keyInputClass?: string
  valueClass?: string
  valueInputClass?: string
  childrenClass?: string
  inputClass?: string
}>(), {
  baseIndent: 20,
  allowKeyEdit: false,
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
    treeData.value[newKey] = 'value';
    emit('update:modelValue', { ...treeData.value });
  }
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
          :node-class="nodeClass"
          :toggle-class="toggleClass"
          :key-class="keyClass"
          :key-input-class="keyInputClass"
          :value-class="valueClass"
          :value-input-class="valueInputClass"
          :children-class="childrenClass"
          :input-class="inputClass"
          @update-value="(val) => updateNode({ ...treeData, [key]: val })"
      >
        <template #toggle-icon="{ expanded }">
          <slot name="toggle-icon" :expanded="expanded" />
        </template>
      </JSONTreeNode>
      <div>
        <button @click="addRow">+ Add new</button>
      </div>
    </ul>
  </div>
</template>