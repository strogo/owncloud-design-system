<template>
  <oc-table-cell type="th" :align-h="alignH" :align-v="alignV" :width="width" class="oc-th">
    <slot />
  </oc-table-cell>
</template>
<script>
import OcTableCell from "./_OcTableCell"

export default {
  name: "OcTh",
  status: "review",
  release: "2.1.0",
  components: { OcTableCell },
  props: {
    alignH: {
      type: String,
      default: "left",
      validator: alignment => /(left|center|right)/.test(alignment),
    },
    alignV: {
      type: String,
      default: "bottom",
      validator: alignment => /(top|middle|bottom)/.test(alignment),
    },
    width: {
      type: String,
      default: "auto",
      validator: width => /(auto|shrink|expand)/.test(width),
    },
  },
}
</script>
<style lang="scss">
.oc-th {
  color: $muted-color;
  font-weight: inherit;
}
</style>
