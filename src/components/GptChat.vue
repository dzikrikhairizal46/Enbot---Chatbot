<template>
  <!-- Komponen utama -->
  <div>
    <!-- Tampilkan chat sambutan dari bot OpenAI -->
    <div class="chat chat-start">
      <div class="bg-gray-500 text-white chat-bubble prose">
        <p>Hello! I'm your TOEFL chatbot. How can I assist you with your TOEFL preparation?</p>
      </div>
    </div>

    <!-- Tampilkan tombol-tombol rekomendasi struktur pembelajaran TOEFL -->
    <div class="chat chat-start mt-4">
      <div class="bg-gray-500 text-white chat-bubble prose">
        <p>Here are some recommended TOEFL learning structures:</p>
        <div class="flex flex-wrap gap-4">
          <!-- Tombol-tombol untuk masing-masing struktur pembelajaran TOEFL -->
          <button class="btn" @click="handleQuery('Reading Comprehension in TOEFL')">Reading Comprehension</button>
          <button class="btn" @click="handleQuery('Listening Comprehension in TOEFL')">Listening Comprehension</button>
          <button class="btn" @click="handleQuery('Structure & Written Expression in TOEFL')">Structure & Written Expression</button>
          <button class="btn" @click="handleQuery('Speaking in TOEFL')">Speaking</button>
        </div>
      </div>
    </div>

    <!-- Tampilkan konten informasi pembelajaran -->
    <div v-if="responseContent.length > 0" class="mt-4 mb-48 p-4 card bg-base-200 shadow-xl">
      <!-- Konten chat -->
      <div v-for="(content, i) in responseContent" :key="i">
        <div v-if="content.role === Role.User" class="chat chat-end">
          <div class="bg-blue-500 text-white chat-bubble">
            {{ content.question }}
          </div>
        </div>
        <div v-else class="chat chat-start">
          <!-- Tampilkan pesan saat loading -->
          <div v-if="isLoading && i + 1 === responseContent.length" class="bg-gray-500 text-white chat-bubble">
            <LoadingText />
          </div>
          <!-- Tampilkan pesan error -->
          <div v-else-if="content.role === Role.Error" class="bg-error text-white chat-bubble">
            {{ content.answer }}
          </div>
          <!-- Tampilkan pesan hasil jawaban -->
          <div v-else class="bg-gray-500 text-white chat-bubble prose">
            <!-- Tampilkan pesan dalam blok kode jika ada -->
            <template v-for="(block, index) in splitContent(content.answer)" :key="index">
              <component v-if="instanceOfCodeBlock(block)" :is="HighlightJs" :code="block.codeBlock" />
              <!-- Tampilkan pesan biasa jika tidak ada blok kode -->
              <p v-else class="mb-2">{{ block }}</p>
            </template>
          </div>
        </div>
      </div>

      <!-- Konten informasi pembelajaran -->
      <div v-if="showLearningContent">
        <div class="chat chat-start">
          <div class="bg-blue-500 text-white chat-bubble">
            {{ selectedLearningTopic }}
          </div>
        </div>
        <div class="chat chat-start">
          <div class="bg-gray-500 text-white chat-bubble prose">
            <p>{{ learningContent }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Input untuk mengajukan pertanyaan -->
    <div class="flex justify-center">
      <div class="w-1/2 fixed bottom-5 fixed bottom-10">
        <input
          v-model="query"
          type="text"
          class="bg-gray-200 w-full border-2 p-4 rounded-lg outline-none"
          placeholder="Ask me anything..."
          @keypress.enter="handleManualQuery"
          autofocus
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  // Import library dan modul yang diperlukan
  import { Configuration, OpenAIApi } from 'openai';
  import { ref, Ref } from 'vue';
  import HighlightJs from './HighlightJs.vue';
  import LoadingText from './LoadingText.vue';

  // Enum untuk peran chat
  enum Role {
    User = 'user',
    Error = 'error',
  }

  // Konfigurasi API Key OpenAI
  const configuration = new Configuration({
    apiKey: import.meta.env.VITE_OPENAI_API_KEY, // Mengambil API Key dari environment variable VITE_OPENAI_API_KEY
  });

  // Variabel-variabel untuk state
  const query: Ref<string> = ref(''); // Pertanyaan yang diajukan oleh pengguna
  const responseContent: Ref<ContentData[]> = ref([]); // Konten respon chat dari OpenAI
  const isLoading: Ref<boolean> = ref(false); // Boolean untuk menunjukkan status loading saat mengirim pertanyaan
  const showLearningContent: Ref<boolean> = ref(false); // Boolean untuk menunjukkan konten informasi pembelajaran
  const selectedLearningTopic: Ref<string> = ref(''); // Topik pembelajaran yang dipilih oleh pengguna
  const learningContent: Ref<string> = ref(''); // Konten informasi pembelajaran

  // Fungsi untuk mengirim pertanyaan ke OpenAI
  const handleQuery: (question: string) => void = async (question) => {
    if (question === '') {
      return;
    }
    isLoading.value = true;

    // Tambahkan pertanyaan pengguna ke daftar konten chat dengan pesan "Loading..."
    responseContent.value.push({
      question,
      role: Role.User,
      answer: 'Loading...',
    });

    // Gulir ke bawah setelah menambahkan pesan
    scrollToBottom();

    // Buat instance dari OpenAI API dengan konfigurasi yang telah dibuat
    const openai: OpenAIApi = new OpenAIApi(configuration);

    try {
      // Kirim pertanyaan ke OpenAI untuk mendapatkan jawaban
      const response = await openai.createChatCompletion({
        model: import.meta.env.VITE_OPENAI_MODEL || 'gpt-3.5-turbo', // Menggunakan model GPT-3.5 Turbo atau model yang ditentukan di environment variable VITE_OPENAI_MODEL
        messages: [{ role: 'user', content: question }],
      });

      // Ambil konten respon dari jawaban
      const content = response.data.choices[0].message as ResponseContentData;

      // Ganti pesan "Loading..." dengan respon dari OpenAI
      responseContent.value.splice(responseContent.value.length - 1, 1, {
        question,
        role: content.role,
        answer: content.content,
      });
    } catch (error: any) {
      // Tangani error jika terjadi
      const errorResponse = error.response.data as ErrorResponse;

      // Tampilkan pesan error dalam konten chat
      responseContent.value.splice(responseContent.value.length - 1, 1, {
        question,
        role: Role.Error,
        answer: errorResponse.error.message,
      });
    } finally {
      // Setelah selesai, reset nilai query dan hentikan status loading
      isLoading.value = false;
      query.value = '';
      // Gulir ke bawah setelah menambahkan respon
      scrollToBottom();
    }
  }

  // Fungsi untuk menampilkan konten informasi pembelajaran
  const showLearningStructure: (topic: string) => void = async (topic) => {
    showLearningContent.value = true; // Set showLearningContent menjadi true untuk menampilkan konten informasi pembelajaran
    selectedLearningTopic.value = topic; // Simpan topik pembelajaran yang dipilih

    try {
      // Buat instance dari OpenAI API dengan konfigurasi yang telah dibuat
      const openai: OpenAIApi = new OpenAIApi(configuration);

      // Kirim pertanyaan ke OpenAI untuk mendapatkan konten informasi pembelajaran
      const response = await openai.createChatCompletion({
        model: import.meta.env.VITE_OPENAI_MODEL || 'gpt-3.5-turbo', // Menggunakan model GPT-3.5 Turbo atau model yang ditentukan di environment variable VITE_OPENAI_MODEL
        messages: [{ role: 'user', content: `Tell me about ${topic} learning structure.` }],
      });

      // Ambil konten respon dari jawaban
      const content = response.data.choices[0].message as ResponseContentData;

      // Simpan konten informasi pembelajaran ke dalam variabel learningContent
      learningContent.value = content.content;
    } catch (error: any) {
      // Tangani error jika terjadi
      const errorResponse = error.response.data as ErrorResponse;

      // Tampilkan pesan error dalam konten chat
      learningContent.value = `An error occurred: ${errorResponse.error.message}`;
    }
  }

  // Fungsi untuk melakukan scrolling ke bagian bawah chat
  const scrollToBottom = (): void => {
    window.scrollTo(0, document.body.scrollHeight);
  }

  // Fungsi untuk memisahkan konten menjadi paragraf atau blok kode
  const splitContent: (content: string) => (string | CodeBlock)[] = (content) => {
    const paragraphs = content.split('\n\n');
    const codeBlocks = content.split('```');

    // Jika terdapat blok kode dalam konten, tampilkan blok kode sebagai komponen HighlightJs
    if (codeBlocks.length > 1) {
      return codeBlocks.map((codeBlock, i) => {
        if (i % 2 === 0) {
          return `${codeBlock}`;
        } else {
          return {
            codeBlock,
          };
        }
      });
    } else {
      // Jika tidak ada blok kode, tampilkan konten sebagai paragraf
      return paragraphs.map((paragraph) => `${paragraph}`);
    }
  }

  // Fungsi untuk memeriksa apakah objek merupakan blok kode
  const instanceOfCodeBlock = (object: any): object is CodeBlock => {
    if (typeof object !== 'object') return false;
    return 'codeBlock' in object;
  }

  // Fungsi untuk mengirim pertanyaan manual melalui chatbox
  const handleManualQuery: () => void = () => {
    if (query.value.trim() !== '') {
      const question = query.value.trim();
      handleQuery(question);
    }
  };
</script>

<style>
  /* Gaya tombol */
  .btn {
    padding: 10px 16px;
    border-radius: 8px;
    background-color: #4caf50;
    color: white;
    font-size: 16px;
    cursor: pointer;
  }

  .btn:hover {
    background-color: #45a049;
  }

  /* Gaya chat bubble */
  .chat {
    margin: 8px 0;
  }

  .chat-bubble {
    padding: 8px;
    border-radius: 8px;
  }

  .chat-start {
    text-align: right;
  }

  .chat-end {
    text-align: left;
  }

  /* Gaya paragraf pada bubble jawaban */
  .chat-bubble p {
    margin-bottom: 0.5rem;
    text-align: left; /* Rata kiri pada bubble jawaban */
  }
</style>
