<template>
  <button>
    <slot />
  </button>
</template>

<style scoped>
button {
  color: var(--iansu-background);
  background-color: var(--iansu-foreground);
  border: 1px solid var(--iansu-foreground);
  border-radius: var(--iansu-radius);
  padding: 6px 12px;
  @apply text-sm;
}
button:hover {
  color: var(--iansu-foreground);
  background-color: var(--iansu-background);
}
</style>
