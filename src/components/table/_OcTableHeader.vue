<template>
  <thead class="oc-thead">
    <slot />
  </thead>
</template>
<script>
export default {
  name: "OcThead",
  status: "review",
  release: "2.1.0",
}
</script>
<style lang="scss">
.oc-thead {
  border-bottom: 1px solid $border-primary;
}
</style>
