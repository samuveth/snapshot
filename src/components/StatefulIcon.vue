<template>
  <button :class="buttonClasses" @click="handleClick($event)">
    <Icon :name="name" size="22" />
  </button>
</template>

<script>
import { computed } from 'vue';
export default {
  props: {
    onName: String,
    offName: String,
    on: { type: Boolean, default: true }
  },
  emits: ['favorite'],
  setup(props, { emit }) {
    const name = computed(() => (props.on ? 'favorite-on' : 'favorite-off'));

    const buttonClasses = computed(() => {
      return {
        'extra-icon': true,
        'extra-icon-off': !props.on,
        'text-primary': props.on,
        'text-gray': !props.on
      };
    });

    function handleClick(e) {
      e.preventDefault();
      emit('favorite');
    }

    return { name, buttonClasses, handleClick };
  }
};
</script>

<style scoped lang="scss">
.extra-icon {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 8px;
  line-height: 16px;
  border: none;
  background-color: transparent;
}
</style>
