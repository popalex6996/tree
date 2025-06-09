<script setup lang="ts">
import { ref } from 'vue'

import type { NodeType } from '@/components/TreeView.vue'

const { node, selectedNodesIds } = defineProps<{ node: NodeType; selectedNodesIds: string[] }>()
defineEmits(['select'])

const expanded = ref(false)

const onExpand = () => {
  expanded.value = !expanded.value
}
</script>

<template>
  <div class="tree-item">
    <div class="optional-row">
      <input
        type="checkbox"
        id="checkbox"
        :checked="selectedNodesIds.includes(node.id)"
        @change="$emit('select', node.id)"
      />
      <label
        for="checkbox"
        @click.prevent="onExpand"
        :class="{ expandable: node.children?.length }"
      >
        <span>
          {{ node.name }}
        </span>
      </label>
      <span v-if="node.children?.length">({{ node.children?.length }})</span>
    </div>

    <div v-if="expanded" class="children">
      <TreeItem
        v-for="child in node.children"
        :key="child.id"
        :node="child"
        :selected-nodes-ids="selectedNodesIds"
        @select="(id: string) => $emit('select', id)"
      />
    </div>
  </div>
</template>

<style scoped>
.tree-item {
  padding: 5px 15px;
  .optional-row {
    gap: 5px;
    display: flex;
    align-items: center;

    #checkbox {
      width: 15px;
      height: 15px;
    }

    .expandable {
      cursor: pointer;
      &:hover {
        color: #42b983;
      }
    }
  }

  .children {
    border-left: 1px solid #42b983;
  }
}
</style>
