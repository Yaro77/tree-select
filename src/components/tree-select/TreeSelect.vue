<script lang="ts">
import {
  defineComponent,
  provide,
  reactive,
  computed,
  toRefs,
  watch,
  watchEffect,
  nextTick,
  PropType,
} from 'vue';
import TreeNode from './TreeNode.vue';
import { SelectionMode, CheckState } from './types';
import {
  ID_FN_KEY,
  TEXT_FN_KEY,
  CHILDREN_FN_KEY,
  SELECTION_MODE_KEY,
} from './constants';

interface State {
  nodesLocal: any[];
  selectedIds: any[];
}

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
    selectionMode: {
      type: String as PropType<SelectionMode>,
      default: SelectionMode.Node,
      validator: (val) => Object.values(SelectionMode).includes(val),
    },
    value: {
      type: Array,
      default: () => [],
    },
  },
  setup(props, context) {
    provide(ID_FN_KEY, props.idFn);
    provide(TEXT_FN_KEY, props.textFn);
    provide(CHILDREN_FN_KEY, props.childrenFn);
    provide(SELECTION_MODE_KEY, props.selectionMode);

    const state = reactive<State>({
      nodesLocal: props.nodes.map((n) => createLocalNode(n)),
      selectedIds: [],
    });

    watch(
      () => state.selectedIds,
      (selectedIds) => console.log('SelectedIds:', selectedIds)
    );

    function filterNodes(tree, childrenFn, predicate) {
      const result = [];

      const methodImpl = (n) => {
        if (predicate(n)) {
          result.push(n);
        }

        const children = childrenFn(n) || [];
        for (const child of children) {
          methodImpl(child.node);
        }
      };

      for (const root of tree) {
        methodImpl(root);
      }
      return result;
    }

    function walkNodes(tree, childrenFn, predicate) {
      const result = [];

      const methodImpl = (n) => {
        if (predicate(n)) {
          result.push(n);
          return;
        }

        const children = childrenFn(n) || [];
        for (const child of children) {
          methodImpl(child.node);
        }
      };

      for (const root of tree) {
        methodImpl(root);
      }
      return result;
    }

    function createLocalNode(n) {
      const children = props.childrenFn(n) || [];
      const initialCheck = CheckState.Unchecked;
      const node = {
        ...n,
        $id: props.idFn(n),
        $check: initialCheck,
        $expanded: true, // todo: default
        children: children.map((ch) => ({
          node: createLocalNode(ch),
          $check:
            initialCheck === CheckState.Checked &&
            props.selectionMode === SelectionMode.Subtree
              ? CheckState.Checked
              : CheckState.Unchecked,
        })),
      };
      return node;
    }

    function onCheckChange() {
      // выполняется в nextTick, так как каскадное изменение $check происходит через prop+watch
      nextTick(() => {
        let checkedNodes = [];
        if (props.selectionMode === SelectionMode.Node) {
          checkedNodes = filterNodes(
            state.nodesLocal,
            props.childrenFn,
            (n) => n.$check === CheckState.Checked
          );

          state.selectedIds = checkedNodes.map((n) => props.idFn(n));
        }
        if (props.selectionMode === SelectionMode.Subtree) {
          checkedNodes = walkNodes(
            state.nodesLocal,
            props.childrenFn,
            (n) => n.$check === CheckState.Checked
          );
        }
        state.selectedIds = checkedNodes.map((n) => props.idFn(n));
      });
    }

    return {
      ...toRefs(state),
      onCheckChange,
    };
  },
});
</script>

<template>
  <div class="tree-select">
    <ul class="tree-select__list">
      <tree-node
        v-for="n of nodesLocal"
        :key="n.$id"
        :node="n"
        @check-change="onCheckChange"
      ></tree-node>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
.tree-select {
  color: #333e48;

  * {
    color: inherit;
  }
}

.tree-select__list {
  list-style-type: none;
  padding-inline-start: 0;
  margin-block: 0;
}
</style>
