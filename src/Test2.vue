<template>
    <div class="vtl">
      <div
        v-if="model.name !== 'root'"
        :id="model.id"
        class="vtl-node"
        :class="{ 'vtl-leaf-node': model.isLeaf, 'vtl-tree-node': !model.isLeaf }"
      >
        <div
          class="vtl-border vtl-up"
          :class="{ 'vtl-active': isDragEnterUp }"
          @drop="dropBefore"
          @dragenter="dragEnterUp"
          @dragover="dragOverUp"
          @dragleave="dragLeaveUp"
        />
        <div
          :class="treeNodeClass"
          :draggable="!model.dragDisabled"
          @dragstart="dragStart"
          @dragover="dragOver"
          @dragenter="dragEnter"
          @dragleave="dragLeave"
          @drop="drop"
          @dragend="dragEnd"
          @mouseover="mouseOver"
          @mouseout="mouseOut"
          @click="click"
        >
          <span
            class="vtl-caret vtl-is-small"
            v-if="model.children && model.children.length > 0"
          >
            <i class="vtl-icon" :class="caretClass" @click.prevent="toggle"></i>
          </span>
  
          <span v-if="model.isLeaf">
            <slot
              name="leafNodeIcon"
              :expanded="expanded"
              :model="model"
              :root="rootNode"
            >
              <i class="vtl-icon vtl-menu-icon vtl-icon-file"></i>
            </slot>
          </span>
          <span v-else>
            <slot
              name="treeNodeIcon"
              :expanded="expanded"
              :model="model"
              :root="rootNode"
            >
              <i class="vtl-icon vtl-menu-icon vtl-icon-folder"></i>
            </slot>
          </span>
  
          <div class="vtl-node-content" v-if="!editable">
            <slot
              name="leafNameDisplay"
              :expanded="expanded"
              :model="model"
              :root="rootNode"
            >
              {{ model.name }}
            </slot>
          </div>
          <input
            v-else
            class="vtl-input"
            type="text"
            ref="nodeInput"
            :value="model.name"
            @input="updateName"
            @blur="setUnEditable"
            @keyup.enter="setUnEditable"
          />
          <div class="vtl-operation" v-show="isHover">
            <span
              :title="defaultAddTreeNodeTitle"
              @click.prevent="addChild(false)"
              v-if="!model.isLeaf && !model.addTreeNodeDisabled"
            >
              <slot
                name="addTreeNodeIcon"
                :expanded="expanded"
                :model="model"
                :root="rootNode"
              >
                <i class="vtl-icon vtl-icon-folder-plus-e"></i>
              </slot>
            </span>
            <span
              :title="defaultAddLeafNodeTitle"
              @click.prevent="addChild(true)"
              v-if="!model.isLeaf && !model.addLeafNodeDisabled"
            >
              <slot
                name="addLeafNodeIcon"
                :expanded="expanded"
                :model="model"
                :root="rootNode"
              >
                <i class="vtl-icon vtl-icon-plus"></i>
              </slot>
            </span>
            <span
              title="edit"
              @click.prevent="setEditable"
              v-if="!model.editNodeDisabled"
            >
              <slot
                name="editNodeIcon"
                :expanded="expanded"
                :model="model"
                :root="rootNode"
              >
                <i class="vtl-icon vtl-icon-edit"></i>
              </slot>
            </span>
            <span
              title="delete"
              @click.prevent="delNode"
              v-if="!model.delNodeDisabled"
            >
              <slot
                name="delNodeIcon"
                :expanded="expanded"
                :model="model"
                :root="rootNode"
              >
                <i class="vtl-icon vtl-icon-trash"></i>
              </slot>
            </span>
          </div>
        </div>
  
        <div
          v-if="model.children && model.children.length > 0 && expanded"
          class="vtl-border vtl-bottom"
          :class="{ 'vtl-active': isDragEnterBottom }"
          @drop="dropAfter"
          @dragenter="dragEnterBottom"
          @dragover="dragOverBottom"
          @dragleave="dragLeaveBottom"
        ></div>
      </div>
  
      <div
        :class="{ 'vtl-tree-margin': model.name !== 'root' }"
        v-show="model.name === 'root' || expanded"
        v-if="isFolder"
      >
        <Test
          v-for="childModel in model.children"
          :default-tree-node-name="defaultTreeNodeName"
          :default-leaf-node-name="defaultLeafNodeName"
          :default-expanded="defaultExpanded"
          :model="childModel"
          :key="childModel.id"
        >
          <template v-for="(_, name) in $slots" #[name]="slotData">
            <slot :name="name" v-bind="slotData" />
          </template>
        </Test>
      </div>
    </div>
  </template>
  
  <script setup>
  import {
    ref,
    computed,
    onBeforeUnmount,
    nextTick,
    getCurrentInstance,
    useSlots,
    toRefs,
  } from "vue";
  import { TreeNode } from "./Tree.js";
  import { removeHandler } from "./tools.js";
  
  const props = defineProps({
    model: {
      type: Object,
      required: true,
    },
    dragStartInfo: {
      type: Object,
    },
    defaultLeafNodeName: {
      type: String,
      default: "Leaf Node",
    },
    defaultTreeNodeName: {
      type: String,
      default: "Tree Node",
    },
    defaultAddTreeNodeTitle: {
      type: String,
      default: "Add Tree Node",
    },
    defaultAddLeafNodeTitle: {
      type: String,
      default: "Add Leaf Node",
    },
    defaultExpanded: {
      type: Boolean,
      default: true,
    },
  });
  
  const { dragStartInfo } = toRefs(props);
  
  const emit = defineEmits([
    "change-name",
    "delete-node",
    "end-edit",
    "click",
    "add-node",
    "drop",
    "drop-before",
    "drop-after",
    "set-drag-start-info",
  ]);
  
  const isHover = ref(false);
  const editable = ref(false);
  const isDragEnterUp = ref(false);
  const isDragEnterBottom = ref(false);
  const isDragEnterNode = ref(false);
  const expanded = ref(props.defaultExpanded);
  const nodeInput = ref(null);
  
  const instance = getCurrentInstance();
  const slots = useSlots();
  
  const rootNode = computed(() => {
    let node = instance.parent;
    while (node.props.model.name !== "root") {
      node = node.parent;
    }
    return node;
  });
  
  const caretClass = computed(() =>
    expanded.value ? "vtl-icon-caret-down" : "vtl-icon-caret-right"
  );
  
  const isFolder = computed(
    () => props.model.children && props.model.children.length
  );
  
  const treeNodeClass = computed(() => ({
    "vtl-node-main": true,
    "vtl-active": isDragEnterNode.value,
    "vtl-drag-disabled": props.model.dragDisabled,
    "vtl-disabled": props.model.disabled,
  }));
  
  const updateName = (e) => {
    const oldName = props.model.name;
    props.model.changeName(e.target.value);
    emit("change-name", {
      id: props.model.id,
      oldName,
      newName: e.target.value,
      node: props.model,
    });
  };
  
  const delNode = () => {
    emit("delete-node", props.model);
  };
  
  const setEditable = () => {
    editable.value = true;
    nextTick(() => {
      const input = nodeInput.value;
      input.focus();
      input.setSelectionRange(0, input.value.length);
    });
  };
  
  const setUnEditable = (e) => {
    if (editable.value === false) return;
    editable.value = false;
    const oldName = props.model.name;
    props.model.changeName(e.target.value);
    emit("change-name", {
      id: props.model.id,
      oldName,
      newName: e.target.value,
      eventType: "blur",
    });
    emit("end-edit", {
      id: props.model.id,
      oldName,
      newName: e.target.value,
    });
  };
  
  const toggle = () => {
    if (isFolder.value) {
      expanded.value = !expanded.value;
    }
  };
  
  const mouseOver = () => {
    if (props.model.disabled) return;
    isHover.value = true;
  };
  
  const mouseOut = () => {
    isHover.value = false;
  };
  
  const click = () => {
    emit("click", {
      toggle,
      ...props.model,
    });
  };
  
  const addChild = (isLeaf) => {
    const name = isLeaf ? props.defaultLeafNodeName : props.defaultTreeNodeName;
    expanded.value = true;
    const node = new TreeNode({ name, isLeaf });
    props.model.addChildren(node, true);
    emit("add-node", node);
  };
  
  const dragStart = (e) => {
    if (!(props.model.dragDisabled || props.model.disabled)) {
      console.log('[ instance ] >', instance)
      emit("set-drag-start-info", instance);
      console.log('[ instance ] >', instance)
  
      e.dataTransfer.setData("data", "data");
      e.dataTransfer.effectAllowed = "move";
      return true;
    }
    return false;
  };
  
  const dragEnd = () => {
    emit("set-drag-start-info", null);
  };
  
  const dragOver = (e) => {
    e.preventDefault();
    return true;
  };
  
  const dragEnter = () => {
    if (!dragStartInfo.value) return;
    if (
      dragStartInfo.value.props.model.id === props.model.id ||
      !dragStartInfo.value ||
      props.model.isLeaf
    )
      return;
    isDragEnterNode.value = true;
  };
  
  const dragLeave = () => {
    isDragEnterNode.value = false;
  };
  
  const drop = () => {
    if (!dragStartInfo.value) return;
    const oldParent = dragStartInfo.value.props.model.parent;
    dragStartInfo.value.props.model.moveInto(props.model);
    isDragEnterNode.value = false;
    emit("drop", {
      target: props.model,
      node: dragStartInfo.value.props.model,
      src: oldParent,
    });
  };
  
  const dragEnterUp = () => {
    if (!dragStartInfo.value) return;
    isDragEnterUp.value = true;
  };
  
  const dragOverUp = (e) => {
    e.preventDefault();
    return true;
  };
  
  const dragLeaveUp = () => {
    if (!dragStartInfo.value) return;
    isDragEnterUp.value = false;
  };
  
  const dropBefore = () => {
    if (!dragStartInfo.value) return;
    const oldParent = dragStartInfo.value.props.model.parent;
    dragStartInfo.value.props.model.insertBefore(props.model);
    isDragEnterUp.value = false;
    emit("drop-before", {
      target: props.model,
      node: dragStartInfo.value.props.model,
      src: oldParent,
    });
  };
  
  const dragEnterBottom = () => {
    if (!dragStartInfo.value) return;
    isDragEnterBottom.value = true;
  };
  
  const dragOverBottom = (e) => {
    e.preventDefault();
    return true;
  };
  
  const dragLeaveBottom = () => {
    if (!dragStartInfo.value) return;
    isDragEnterBottom.value = false;
  };
  
  const dropAfter = () => {
    if (!dragStartInfo.value) return;
    const oldParent = dragStartInfo.value.props.model.parent;
    dragStartInfo.value.props.model.insertAfter(props.model);
    isDragEnterBottom.value = false;
    emit("drop-after", {
      target: props.model,
      node: dragStartInfo.value.props.model,
      src: oldParent,
    });
  };
  
  onBeforeUnmount(() => {
    removeHandler(window, "keyup");
  });
  </script>
  
  <style lang="less">
  @font-face {
    font-family: "icomoon";
    src: url("fonts/icomoon.eot?ui1hbx");
    src: url("fonts/icomoon.eot?ui1hbx#iefix") format("embedded-opentype"),
      url("fonts/icomoon.ttf?ui1hbx") format("truetype"),
      url("fonts/icomoon.woff?ui1hbx") format("woff"),
      url("fonts/icomoon.svg?ui1hbx#icomoon") format("svg");
    font-weight: normal;
    font-style: normal;
  }
  
  .vtl-icon {
    font-family: "icomoon" !important;
    speak: none;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
  
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    &.vtl-menu-icon {
      margin-right: 4px;
      &:hover {
        color: inherit;
      }
    }
    &:hover {
      color: blue;
    }
  }
  
  .vtl-icon-file:before {
    content: "\e906";
  }
  .vtl-icon-folder:before {
    content: "\e907";
  }
  .vtl-icon-caret-down:before {
    content: "\e901";
  }
  .vtl-icon-caret-right:before {
    content: "\e900";
  }
  .vtl-icon-edit:before {
    content: "\e902";
  }
  .vtl-icon-folder-plus-e:before {
    content: "\e903";
  }
  .vtl-icon-plus:before {
    content: "\e904";
  }
  .vtl-icon-trash:before {
    content: "\e905";
  }
  
  .vtl-border {
    height: 5px;
    &.vtl-up {
      margin-top: -5px;
      background-color: transparent;
    }
    &.vtl-bottom {
      background-color: transparent;
    }
    &.vtl-active {
      border-bottom: 3px dashed blue;
    }
  }
  
  .vtl-node-main {
    display: flex;
    align-items: center;
    padding: 5px 0 5px 1rem;
    .vtl-input {
      border: none;
      max-width: 150px;
      border-bottom: 1px solid blue;
    }
    &:hover {
      background-color: #f0f0f0;
    }
    &.vtl-active {
      outline: 2px dashed pink;
    }
    .vtl-caret {
      margin-left: -1rem;
    }
    .vtl-operation {
      margin-left: 2rem;
      letter-spacing: 1px;
    }
  }
  
  .vtl-item {
    cursor: pointer;
  }
  .vtl-tree-margin {
    margin-left: 2em;
  }
  </style>
  