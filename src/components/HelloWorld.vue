<script setup>
import { ref } from 'vue';
import { TreeSelect } from './tree-select';

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
</script>

<template>
  <tree-select
    :nodes="nodes"
    :id-fn="(n) => n.id"
    :text-fn="(n) => n.text"
    :children-fn="(n) => n.children"
  ></tree-select>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
