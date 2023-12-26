<script setup lang="ts">
  // Import library dan modul yang diperlukan
  import 'highlight.js/lib/common'; // Import library highlight.js untuk mendukung penyorotan kode
  import hljsVuePlugin from '@highlightjs/vue-plugin'; // Import plugin untuk mengintegrasikan highlight.js dengan Vue
  import { defineComponent } from 'vue'; // Import defineComponent dari Vue untuk mendefinisikan komponen Vue

  // Menggunakan komponen HighlightJs dari plugin highlight.js untuk syntax highlighting
  const HighlightJs = hljsVuePlugin.component;

  // Definisikan komponen Vue menggunakan defineComponent()
  defineComponent({
    components: {
      HighlightJs, // Daftarkan komponen HighlightJs sehingga dapat digunakan di template
    },
  });

  // Definisikan properti 'code' sebagai input yang dibutuhkan oleh komponen
  defineProps({
    code: {
      type: String,
      required: true,
    },
  });
</script>

<template>
  <!-- Gunakan komponen HighlightJs untuk menyorot kode -->
  <HighlightJs autodetect :code="code" />
</template>
