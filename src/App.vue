<template>
  <el-tabs>
    <el-tab-pane label="Partial Collection" lazy>
      <MultipleCollection/>
    </el-tab-pane>
    <el-tab-pane label="Single Collection" lazy>
      <SingleCollection/>
    </el-tab-pane>
  </el-tabs>
</template>

<script setup lang="ts">


import SingleCollection from "./components/SingleCollection.vue";
import MultipleCollection from "./components/MultipleCollection.vue";
</script>
