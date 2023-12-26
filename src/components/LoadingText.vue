<script setup lang="ts">
  // Import library dan modul yang diperlukan dari Vue
  import { onBeforeUnmount, onMounted, ref, Ref } from 'vue';

  // Buat ref untuk menyimpan teks yang akan ditampilkan
  const loadingText: Ref<string> = ref('Loading');

  // Buat ref untuk menyimpan ID interval yang digunakan untuk mengganti teks secara berkala
  // Gunakan tipe Ref<number | NodeJS.Timeout> karena interval.value bisa berupa number atau NodeJS.Timeout
  const interval: Ref<number | NodeJS.Timeout> = ref(0);

  // Fungsi yang akan dijalankan ketika komponen selesai dimount
  onMounted(() => {
    // Daftar teks yang akan ditampilkan secara bergantian
    const loadingTexts = ['Loading', 'Loading.', 'Loading..', 'Loading...'];

    // Variabel untuk menyimpan index teks yang sedang ditampilkan
    let i = 0;

    // Mulai interval untuk mengganti teks setiap 500 milidetik
    interval.value = setInterval(() => {
      loadingText.value = loadingTexts[i];
      i = (i + 1) % loadingTexts.length; // Gunakan modulo untuk menghindari indeks melampaui panjang array
    }, 500);
  });

  // Fungsi yang akan dijalankan sebelum komponen diunmount
  onBeforeUnmount(() => {
    // Hentikan interval yang sedang berjalan agar tidak terjadi memory leak
    clearInterval(interval.value as number); // Perlu konversi tipe menjadi number karena clearInterval memerlukan nilai number
  });
</script>

<template>
  <!-- Tampilkan teks loading yang akan berubah-ubah -->
  <div>{{ loadingText }}</div>
</template>
