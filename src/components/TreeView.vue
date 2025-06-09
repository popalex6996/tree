<script setup lang="ts">
import { ref, computed } from 'vue'

import TreeItem from '@/components/TreeItem.vue'

export type NodeType = {
  id: string
  name: string
  children?: NodeType[]
}

const NODES: NodeType[] = [
  {
    id: '687086d9-e6b9-40a2-a226-5834c67a781d',
    name: 'Europe',
    children: [
      {
        id: '487086d9-e6b9-40a2-a226-5834c67a781d',
        name: 'Ukraine',
        children: [
          {
            id: '9a434c91-13ee-4e19-94c1-47b2f95d7461',
            name: 'Kharkivska',
            children: [],
          },
          {
            id: '7160291a-3da3-418b-8860-bc3b3d058e26',
            name: 'Kyivska',
            children: [],
          },
          {
            id: 'fac202c4-75de-45f6-b850-0d5c3e897964',
            name: 'Vinnytska',
            children: [],
          },
          {
            id: 'e3b3e640-809f-47ba-bf66-5aeea1c3f3ac',
            name: 'Lvivska',
            children: [],
          },
        ],
      },
      {
        id: 'e0a7380f-faec-4e4f-a915-35108857685d',
        name: 'Great britain',
        children: [],
      },
      {
        id: '46611bd2-a5e3-4d2f-89d9-bca6fafcf43b',
        name: 'France',
        children: [],
      },
      {
        id: '78d0d634-380b-4979-9fc0-08307ed6e3d7',
        name: 'Norway',
        children: [],
      },
    ],
  },
  {
    id: '687086ea-e6b9-40a2-a226-5834c67a781d',
    name: 'Asia',
    children: [
      {
        id: '868a248d-26df-433a-aecc-3c004e3c66d3',
        name: 'China',
        children: [],
      },
      {
        id: 'b602579b-5faf-454d-b84a-5ef6605e98fe',
        name: 'Taiwan',
        children: [],
      },
      {
        id: 'fa71bbd5-a052-43cc-bd9d-42a252154ca0',
        name: 'Japan',
        children: [],
      },
      {
        id: 'a0bb0c79-9da2-4ff8-89f7-f55e9a17d69e',
        name: 'Vietnam',
        children: [],
      },
    ],
  },
]

const selectedNodesIds = ref<string[]>([])
const search = ref<string>()

const searchTree = (nodes: NodeType[], searchName: string): NodeType[] => {
  function recurse(node: NodeType): NodeType | null {
    const matchedChildren: NodeType[] = []

    for (const child of node.children) {
      const result = recurse(child)
      if (result) {
        matchedChildren.push(result)
      }
    }

    const isMatch = node.name.toLowerCase().startsWith(searchName.toLowerCase())

    if (isMatch || matchedChildren.length > 0) {
      return {
        id: node.id,
        name: node.name,
        children: matchedChildren,
      }
    }

    return null
  }

  const filtered: NodeType[] = []

  for (const root of nodes) {
    const result = recurse(root)
    if (result) filtered.push(result)
  }

  return filtered
}

const filteredNodes = computed(() => {
  if (!search.value) return NODES
  return searchTree(NODES, search.value)
})

const findNode = (id: string, arr: NodeType[]): NodeType | null => {
  for (const item of arr) {
    if (item.id === id) return item
    if (item.children) {
      const found = findNode(id, item.children)
      if (found) return found
    }
  }
  return null
}

const getChildrenIds = (node: NodeType): string[] => {
  if (!node.children?.length) return []
  return node.children.reduce<string[]>((acc, child) => {
    acc.push(child.id)
    acc.push(...getChildrenIds(child))
    return acc
  }, [])
}

const onSelect = (nodeId: string) => {
  const selectedNode = findNode(nodeId, NODES)
  if (!selectedNode) return
  const childrenIds = getChildrenIds(selectedNode)

  if (selectedNodesIds.value.includes(selectedNode.id)) {
    selectedNodesIds.value = selectedNodesIds.value.filter(
      (id) => id !== selectedNode.id && !childrenIds.includes(id),
    )
  } else {
    selectedNodesIds.value = [selectedNode.id, ...childrenIds, ...selectedNodesIds.value]
  }
}
</script>

<template>
  <div class="tree-view-container">
    <div class="search-bar">
      <label class="search-label" for="search">
        <input name="search" id="search" type="text" v-model="search" placeholder="Search" />
        <div v-for="nodeID in selectedNodesIds" :key="nodeID" class="chip">
          {{ findNode(nodeID, NODES)?.name }}
          <div class="cross" @click="() => onSelect(nodeID)">✕</div>
        </div>
      </label>
    </div>

    <div class="tree-view">
      <TreeItem
        v-for="node in filteredNodes"
        :key="node.id"
        :node="node"
        :selected-nodes-ids="selectedNodesIds"
        @select="onSelect"
      />
    </div>

    <div class="result">categories: {{ selectedNodesIds }}</div>
  </div>
</template>

<style scoped>
.tree-view-container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 10px;
}
.search-bar {
  .search-label {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 5px;
    border: 1px solid cadetblue;
    border-radius: 5px;
    padding: 5px 10px;
    height: auto;
    flex-wrap: wrap;

    #search {
      border: none;
      outline: none;
    }

    .chip {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 2px 5px;
      border-radius: 50px;
      gap: 5px;
      background: cadetblue;
      color: white;

      .cross {
        padding: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 100%;
        color: cadetblue;
        background: white;
        width: 3px;
        height: 3px;
      }
    }
  }
}

.tree-view {
  display: flex;
  align-items: flex-start;
  justify-content: flex-start;
  flex-direction: column;

  padding: 10px 20px;
  border: 1px solid cadetblue;
  border-radius: 5px;
  min-height: 50vw;
  min-width: 50vw;
}

.result {
  min-height: 10vw;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;

  padding: 0 50px;
}
</style>
