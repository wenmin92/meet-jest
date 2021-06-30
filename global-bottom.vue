<template>
  <footer>
    <div
      class="absolute bottom-2 left-3"
      v-if="$slidev.nav.currentLayout !== 'cover'"
    >
      {{ $slidev.nav.currentPage - 1 }} / {{ $slidev.nav.total - 1 }}
    </div>
    <div class="absolute bottom-2.5 right-2">
      <img class="w-24" src="/logo.png" />
    </div>
  </footer>
</template>