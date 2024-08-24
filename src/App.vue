<template>
  <Test
    @click="onClick"
    @change-name="onChangeName"
    @delete-node="onDel"
    @add-node="onAddNode"
    :model="data"
    default-tree-node-name="new node"
    default-leaf-node-name="new leaf"
    v-bind:default-expanded="false"
  >
    <template v-slot:leafNameDisplay="slotProps">
      <span>
        {{ slotProps.model.name }}
        <span class="muted">#{{ slotProps.model.id }}</span>
      </span>
    </template>
    <span class="icon" slot="addTreeNodeIcon">📂</span>
    <span class="icon" slot="addLeafNodeIcon">＋</span>
    <span class="icon" slot="editNodeIcon">📃</span>
    <span class="icon" slot="delNodeIcon">✂️</span>
    <span class="icon" slot="leafNodeIcon">🍃</span>
    <span class="icon" slot="treeNodeIcon">🌲</span>
  </Test>
</template>

<script setup>
import Test from "./Test.vue";
import { ref } from "vue";
import { Tree } from "./Tree.js";
const data = ref(
  new Tree([
    {
      name: "Node 1",
      id: 1,
      pid: 0,
      dragDisabled: true,
      addTreeNodeDisabled: true,
      addLeafNodeDisabled: true,
      editNodeDisabled: true,
      delNodeDisabled: true,
      children: [
        {
          name: "Node 1-2",
          id: 2,
          isLeaf: true,
          pid: 1,
        },
      ],
    },
    {
      name: "Node 2",
      id: 3,
      pid: 0,
      disabled: true,
    },
    {
      name: "Node 3",
      id: 4,
      pid: 0,
    },
  ])
);

function onDel(node) {
  node.remove();
}
</script>

<style lang="less">
.vtl {
  .vtl-drag-disabled {
    background-color: #d0cfcf;
    &:hover {
      background-color: #d0cfcf;
    }
  }
  .vtl-disabled {
    background-color: #d0cfcf;
  }
}
</style>

<style lang="less" scoped>
.icon {
  &:hover {
    cursor: pointer;
  }
}

.muted {
  color: gray;
  font-size: 80%;
}
</style>
