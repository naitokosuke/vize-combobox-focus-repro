<script setup lang="ts">
const listId = "fruits";
</script>

<template>
  <input
    aria-activedescendant="fruit-0"
    :aria-controls="listId"
    aria-expanded="true"
    aria-label="Fruit"
    role="combobox"
  />
  <ul :id="listId" role="listbox">
    <li id="fruit-0" aria-selected="true" role="option">Apple</li>
  </ul>
</template>
