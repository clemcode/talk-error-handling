<template>
  <div class="slidev-layout">
    <slot />
    <div class="split">
      <div>
        <slot name="left" />
      </div>
      <div>
        <slot name="right" />
      </div>
    </div>
  </div>
</template>

<style>
.split {
  height: 100%;
  @apply grid grid-cols-2 gap-8;
}
</style>
