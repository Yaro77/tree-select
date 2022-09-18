<script lang="ts">
import {
  defineComponent,
  inject,
  computed,
  reactive,
  toRefs,
  watch,
  onMounted,
} from 'vue';
import {
  ID_FN_KEY,
  TEXT_FN_KEY,
  CHILDREN_FN_KEY,
  SELECTION_MODE_KEY,
} from './constants';
import { SelectionMode, SelectionState } from './types';

enum CheckState {
  Unchecked,
  Intermediate,
  Checked,
}

interface Child {
  node: any;
  check: CheckState;
}

interface State {
  expanded: boolean;
  check: CheckState;
  children: Child[];
}

export default defineComponent({
  name: 'tree-node',
  props: {
    node: {
      type: Object,
    },
    selected: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['select'],
  setup(props, context) {
    const { emit } = context;
    const { selected } = toRefs(props);
    const idFn = inject<Function>(ID_FN_KEY);
    const textFn = inject<Function>(TEXT_FN_KEY);
    const childrenFn = inject<Function>(CHILDREN_FN_KEY);
    const selectionMode = inject<SelectionMode>(SELECTION_MODE_KEY);

    const childrenCheck =
      selectionMode === SelectionMode.Subtree
        ? selected.value
          ? CheckState.Selected
          : CheckState.Unchecked
        : CheckState.Unchecked;

    const state = reactive<State>({
      expanded: true, // todo: false by default
      check: selected.value ? CheckState.Checked : CheckState.Unchecked,
      children: childrenFn(props.node).map((child) => ({
        node: child,
        check: childrenCheck,
      })),
    });

    onMounted(() => {
      emitSelect(); // update check tracking in parent
    });

    const checkIcon = computed(() => {
      switch (state.check) {
        case CheckState.Unchecked:
          return '#not-selected';
        case CheckState.Intermediate:
          return '#any-child-selected';
        case CheckState.Checked:
          return '#selected';
      }
    });

    watch(selected, (value) => {
      state.ckeck = selectionMode === SelectionMode.Subtree ? value ? CheckState.Checked : CheckState.Unchecked;
      if (
        selectionMode === SelectionMode.Subtree &&
        selection !== Selection.Intermediate
      ) {
        state.children.forEach((ch) => (ch.selection = selection));
      }
    });

    function toggleExpandState() {
      state.expanded = !state.expanded;
    }

    function toggleSelectState() {
      const newSelection =
        state.selection === Selection.Unselected ||
        state.selection === Selection.Intermediate
          ? Selection.Selected
          : Selection.Unselected;
      state.selection = newSelection;
      if (selectionMode === SelectionMode.Subtree) {
        state.children.forEach((ch) => (ch.selection = newSelection));
      }
      emitSelect();
    }

    function onSelectChild({ id, selection }) {
      const child = state.children.find((ch) => id === idFn(ch.node));
      if (!child) {
        throw Error('Child not found, invalid state!');
      }
      child.selection = selection;
      updateSelection();
      emitSelect();
    }

    function updateSelection() {
      if (
        selectionMode === SelectionMode.Subtree &&
        state.children.length > 0
      ) {
        const everyChildSelected = state.children.every(
          (ch) => ch.selection === Selection.Selected
        );
        const everyChildUnselected = state.children.every(
          (ch) => ch.selection === Selection.Unselected
        );
        const text = textFn(props.node);
        if (everyChildSelected) {
          console.log('every selected', text);
          state.selection = Selection.Selected;
        } else if (everyChildUnselected) {
          console.log('every unselected', text);
          state.selection = Selection.Unselected;
        } else {
          console.log('some some', text);
          state.selection = Selection.Intermediate;
        }
      }
    }

    function emitSelect() {
      emit('select', {
        id: idFn(props.node),
        check: state.check,
      });
    }

    function fixChildSelection(selection: Selection) {
      return selection === Selection.Selected
        ? Selection.Selected
        : Selection.Unselected;
    }

    return {
      ...toRefs(state),
      idFn,
      textFn,
      selectionIcon,
      toggleExpandState,
      toggleSelectState,
      onSelectChild,
      fixChildSelection,
    };
  },
});
</script>

<template>
  <li
    class="tree-node"
    :class="{
      'tree-node_expanded': expanded,
      'tree-node_collapsed': !expanded,
    }"
    @click.stop="toggleSelectState"
  >
    <div class="tree-node__element">
      <svg
        width="20"
        height="20"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <use :xlink:href="selectionIcon" />
        <defs>
          <g id="not-selected">
            <rect
              x=".5"
              y=".5"
              width="19"
              height="19"
              rx="1.5"
              fill="#fff"
              stroke="#C4C4C4"
            />
          </g>
          <g id="selected">
            <rect
              x=".5"
              y=".5"
              width="19"
              height="19"
              rx="1.5"
              fill="#fff"
              stroke="#0D77DC"
            />
            <path
              d="M13.494 6.758a.882.882 0 0 1 1.248 1.248L9.447 13.3a.882.882 0 0 1-1.247 0l-2.942-2.94A.882.882 0 1 1 6.506 9.11l2.318 2.318 4.67-4.67Z"
              fill="#0D77DC"
            />
          </g>
          <g id="any-child-selected">
            <rect
              x=".5"
              y=".5"
              width="19"
              height="19"
              rx="1.5"
              fill="#fff"
              stroke="#0D77DC"
            />
            <rect x="4" y="4" width="12" height="12" rx="1" fill="#0D77DC" />
          </g>
        </defs>
      </svg>
      <slot>
        <div class="tree-node__content">
          <div>{{ textFn(node) }}</div>
          <button
            v-if="children.length > 0"
            class="tree-node__expand-collapse"
            @click.stop="toggleExpandState"
          >
            <svg
              width="12"
              height="12"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <g stroke="#C4C4C4">
                <rect x=".5" y=".5" width="11" height="11" rx=".5" />
                <path v-if="expanded" d="M9 6H3" />
                <path v-else d="M6 3v6M9 6H3" />
              </g>
            </svg>
          </button>
        </div>
      </slot>
    </div>
    <ul
      v-if="children.length > 0"
      v-show="expanded"
      class="tree-node__children"
    >
      <tree-node
        v-for="child of children"
        :key="idFn(child.node)"
        :node="child.node"
        :selection="fixChildSelection(child.selection)"
        @select="onSelectChild"
      ></tree-node>
    </ul>
  </li>
</template>

<style lang="scss" scoped>
.tree-node {
  &.tree-node_collapsed {
  }

  &.tree-node_expanded {
  }

  + .tree-node {
    margin-top: 10px;
  }
}

.tree-node__element {
  user-select: none;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  min-height: 20px;

  > :nth-child(1) {
    flex: 0 0 20px;
    margin-inline-end: 8px;
  }

  > :nth-child(2) {
    flex: 1 1 100%;
    min-width: 1px;
  }

  * {
    user-select: inherit;
  }
}

.tree-node__content {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  min-height: 20px;

  > :nth-child(1) {
    flex: 1 1 100%;
    min-width: 1px;
  }

  > :nth-child(2) {
    flex: 0 0 auto;
  }
}

.tree-node__children {
  list-style: none;
  padding-inline-start: 28px;
  margin-block: 10px 0;
}

.tree-node__expand-collapse {
  appearance: none;
  outline: none;
  border: none;
  background: transparent;
  min-height: 20px;
  width: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}
</style>
