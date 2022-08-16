<template>
  <div>
    <div class="split">
      <slot />
    </div>
    <div class="absolute bottom-4 right-4">
      <img src="../images/is-logo-black.png" width="40" />
    </div>
  </div>
</template>

<style>
.split {
  height: 100%;
  @apply grid grid-cols-2 gap-8;
}
</style>
