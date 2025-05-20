<script setup lang="ts">
import { defineProps, defineEmits, ref, watch } from 'vue';
import JSONTreeNode from "./JSONTreeNode.vue";

const props = withDefaults(defineProps<{
    modelValue: Record<string, any> | null;
    baseIndent?: number;
    allowKeyEdit?: boolean;
}>(), {
    baseIndent: 20,
    allowKeyEdit: true,
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
</script>

<style scoped>
.json-tree-editor {
    font-family: monospace;
    padding: 10px;
}
</style>

<template>
    <div class="json-tree-editor">
        <ul class="tree-root">
            <JSONTreeNode
                v-for="(value, key) in treeData"
                :key="key"
                :nodeKey="key"
                :nodeValue="value"
                :baseIndent="baseIndent"
                :allowKeyEdit="allowKeyEdit"
                @update-value="(val) => updateNode({ ...treeData, [key]: val })"
            />
        </ul>
    </div>
</template>
