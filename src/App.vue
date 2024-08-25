<template>
  <Test
    @click="onClick"
    @change-name="onChangeName"
    @delete-node="onDel"
    @add-node="onAddNode"
    @set-drag-start-info="setDragStartInfo"
    :model="data"
    :dragStartInfo="dragStartInfo"
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
const dragStartInfo = ref(null);

const setDragStartInfo = (val) => {
  dragStartInfo.value = val;
};
const data = ref(
  new Tree([
    {
      name: "Node 1",
      id: 1,
      pid: 0,
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
    },
    {
      name: "Node 3",
      id: 4,
      pid: 0,
    },
  ])
);
function onChangeName() {
  console.log(12312312);
}

function onDel(node) {
  console.log('[ 123123 ] >', 123123)
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
