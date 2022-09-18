<script setup>
import { ref } from 'vue';
import { TreeSelect, SelectionMode } from './tree-select';

defineProps({
  msg: String,
});

const nodes = ref([
  {
    id: 'SgJ9m',
    text: 'Users',
    children: [
      {
        id: 'FFfEx',
        text: 'usm',
      },
      {
        id: 'xPOCm',
        text: 'kpt',
        children: [
          {
            id: '3fGcP',
            text: 'Documents',
          },
          {
            id: 'DRvjU',
            text: 'Downloads',
          },
        ],
      },
    ],
  },
  {
    id: 'T2RYy',
    text: 'Windows',
    children: [
      {
        id: 'HPG0D',
        text: 'System32',
      },
    ],
  },
]);

function getId(n) {
  return n.id;
}

function getText(n) {
  return n.text;
}

function getChildren(n) {
  return n.children || [];
}
</script>

<template>
  <tree-select
    :nodes="nodes"
    :id-fn="getId"
    :text-fn="getText"
    :children-fn="getChildren"
    :selection-mode="SelectionMode.Subtree"
  ></tree-select>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
