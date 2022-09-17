<script lang="ts">
import { defineComponent, inject, computed, ref } from 'vue';
import { ID_FN_KEY, TEXT_FN_KEY, CHILDREN_FN_KEY } from './constants';

export default defineComponent({
  name: 'tree-node',
  props: {
    node: {
      type: Object,
    },
  },
  setup(props, context) {
    const idFn = inject(ID_FN_KEY);
    const textFn = inject(TEXT_FN_KEY);
    const childrenFn = inject(CHILDREN_FN_KEY);

    const expanded = ref(false);
    const selected = ref(false);

    const children = computed(() => {
      return childrenFn(props.node) || [];
    });

    const selectionIcon = computed(() => {
      if (selected.value) {
        return '#selected';
      }
      // todo: what about partially selected?
      return '#not-selected';
    });

    function toggleExpandState() {
      expanded.value = !expanded.value;
    }

    function toggleSelectState() {
      selected.value = !selected.value; // todo: select hierarchy, or apply selection strategy
    }

    return {
      idFn,
      textFn,
      expanded,
      children,
      selectionIcon,
      toggleExpandState,
      toggleSelectState,
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
        v-for="node of children"
        :key="idFn(node)"
        :node="node"
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
}

.tree-node__element {
  user-select: none;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  height: 38px;

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
  height: 38px;

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
  margin-block: 0;
}

.tree-node__expand-collapse {
  appearance: none;
  outline: none;
  border: none;
  background: transparent;
  height: 38px;
  width: 38px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}
</style>
