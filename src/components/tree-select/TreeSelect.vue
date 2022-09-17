<script lang="ts">
import { defineComponent, provide } from 'vue';
import TreeNode from './TreeNode.vue';
import { ID_FN_KEY, TEXT_FN_KEY, CHILDREN_FN_KEY } from './constants';

export default defineComponent({
  components: {
    TreeNode,
  },
  props: {
    nodes: {
      type: Array,
      default: () => [],
    },
    idFn: Function,
    textFn: Function,
    childrenFn: Function,
  },
  setup(props, context) {
    provide(ID_FN_KEY, props.idFn);
    provide(TEXT_FN_KEY, props.textFn);
    provide(CHILDREN_FN_KEY, props.childrenFn);
    return {};
  },
});
</script>

<template>
  <div class="tree-select">
    <ul class="tree-select__list">
      <tree-node
        v-for="node of nodes"
        :key="idFn(node)"
        :node="node"
      ></tree-node>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
.tree-select {
}

.tree-select__list {
  list-style-type: none;
  padding-inline-start: 0;
  margin-block: 0;
}
</style>