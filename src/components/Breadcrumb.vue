<template>
  <el-breadcrumb separator="/">
    <el-breadcrumb-item v-for="(b, index) in breadcrumbList" :key="index">{{b}}</el-breadcrumb-item>
</el-breadcrumb>
</template>

<script>
export default {
  name: 'Breadcrumb',
  props: ['breadcrumbList']
}
</script>

<style>

</style>