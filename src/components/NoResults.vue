<template>
  <div class="mb-3 text-center">
    <Block v-if="block" class="pt-1">
      {{ text }}
    </Block>
    <div v-else>{{ text }}</div>
  </div>
</template>

<script>
export default {
  props: ['block'],
  computed: {
    text() {
      return this.$t('noResultsFound');
    }
  }
};
</script>
