<template>
  <li>
    <span @click="$emit('select', node)">
      <i class="fas fa-user-tie" :class="{'act': node.activated, 'aff': node.affiliated}"></i>
      <i class="fas fa-gem" :class="node.rank"></i> <br>
      {{ node.name }} <br>
      <template v-if="depth == 0">
        <p>total: {{ node.closed_points_arr }}</p>
      </template>
      <template v-else>
        <small>compras: {{ node.points }}</small> <br>
        <small v-if="node.affiliation_points">afill: {{ node.affiliation_points }}</small>
      </template>

      <button
        v-if="hasChildren"
        type="button"
        class="tree-toggle"
        :title="expanded ? 'Ocultar' : 'Mostrar ' + node._childs.length + ' integrante(s)'"
        @click.stop="expanded = !expanded"
      >{{ expanded ? '−' : '+' }}</button>
    </span>

    <ul v-if="hasChildren && expanded">
      <tree-node
        v-for="child in node._childs"
        :key="child.id"
        :node="child"
        :depth="depth + 1"
        @select="$emit('select', $event)"
      />
    </ul>
  </li>
</template>

<script>
// Niveles visibles al ingresar: raíz + 3 niveles
const INITIAL_DEPTH = 3

export default {
  name: 'TreeNode',
  props: {
    node:  { type: Object, required: true },
    depth: { type: Number, default: 0 },
  },
  data() {
    return {
      expanded: this.depth < INITIAL_DEPTH,
    }
  },
  computed: {
    hasChildren() {
      return Array.isArray(this.node._childs) && this.node._childs.length > 0
    },
  },
}
</script>

<style lang="stylus">
  .tree span .tree-toggle
    position absolute
    top -6px
    right -6px
    width 22px
    height 22px
    padding 0
    border 0
    border-radius 6px
    background #1976f2
    color white
    font-size 16px
    font-weight 700
    line-height 22px
    cursor pointer
    z-index 1

    &:hover
      background #1263cc
</style>
