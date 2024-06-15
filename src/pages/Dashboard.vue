<template>
  <div class="flex h-screen overflow-hidden">

    <!-- Sidebar -->
    <Sidebar :sidebarOpen="sidebarOpen" @close-sidebar="sidebarOpen = false" />

    <!-- Content area -->
    <div class="relative flex flex-col flex-1 overflow-y-auto overflow-x-hidden">

      <main>
        <div class="px-4 sm:px-6 lg:px-8 py-8 w-full max-w-9xl mx-auto">

          <!-- Welcome banner -->
          <WelcomeBanner />

          <!-- Dashboard actions -->
          <div class="sm:flex sm:justify-between sm:items-center mb-8">

            <!-- Right: Actions -->
            <div class="grid grid-flow-col sm:auto-cols-max justify-start sm:justify-end gap-2">
              <h2 class="text-2x2 font-semibold text-slate-800 px-4 py-2 cursor-pointer inline-block">Your recordings</h2>

              <!-- Add view button -->
              <label class="btn bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 cursor-pointer inline-block">
                Add script
                <input type="file" @change="handleFileUpload" class="hidden" />
              </label>
            </div>
          </div>

          <div class="m-4">
            <div
              class="border border-dashed border-gray-400 p-10 w-full flex align-center flex-col justify-center items-center   max-w-9xl mx-auto p-4">
              <button @click="toggleRecording" :class="isRecording ? 'bg-red-500' : 'bg-green-500'"
                class="text-white py-2 px-4 rounded">
                {{ isRecording ? 'Stop Recording' : 'Start Recording' }}
              </button>
              <div class="text-sm text-gray-500 text-center">
                {{ formatTime(recordingTime) }}
              </div>
            </div>
            
            <span
              class="text-black font-semibold text-xl m-2 text-center block before:content-['-'] after:content-['-'] before:mr-2 after:ml-2">
              Or
            </span>

            <div
              class="drag-drop-area border border-dashed border-gray-400 p-10 w-full flex align-center flex-col justify-center items-center space-x-4  max-w-9xl mx-auto p-4"
              @dragover.prevent @dragenter.prevent="handleDragEnter" @dragleave="handleDragLeave" @drop="handleDrop">
              <p v-if="!isDragging">Drag or drop file here to update</p>
              <p v-else>Drop the file now...</p>
            </div>
          </div>

          <audio v-if="audioUrl" :src="audioUrl" controls class="mt-4 w-full"></audio>

          <div v-else class="text-center text-xl">
            You have no recordings yet <br>
            <span class="text-black font-semibold">Start by making a recording.</span>
          </div>

        </div>
      </main>
    </div>

  </div>
</template>

 
<script>
import { ref } from 'vue'
import Sidebar from '../partials/Sidebar.vue'
import Header from '../partials/Header.vue'
import WelcomeBanner from '../partials/dashboard/WelcomeBanner.vue'

export default {
  name: 'Dashboard',
  components: {
    Sidebar,
    Header,
    WelcomeBanner,
  },

  setup() {
    const sidebarOpen = ref(false)
    return {
      sidebarOpen,
      mediaRecorder: ref(null),
      audioChunks: [],
      audioUrl: ref(null),
      isRecording: ref(false),
      isDragging: ref(false),
      recordingTime: ref(0)
    }
  },

  methods: {
    handleFileUpload() {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          this.audioUrl = reader.result;
        };
        reader.readAsDataURL(file);
      }
    },

    toggleRecording() {
      this.isRecording ? this.stopRecording() : this.startRecording();
    },
    startTimer() {
      this.recordingTime = 0;
      this.timer = setInterval(() => {
        this.recordingTime++;
      }, 1000);
    },
    stopTimer() {
      clearInterval(this.timer);
      this.timer = null;
    },
    formatTime(seconds) {
      const minutes = Math.floor(seconds / 60);
      const secs = seconds % 60;
      return `${minutes}:${secs < 10 ? '0' : ''}${secs}`;
    },
    beforeDestroy() {
      if (this.timer) {
        clearInterval(this.timer);
      }
    },
    async startRecording() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      this.mediaRecorder = new MediaRecorder(stream);
      this.mediaRecorder.start();
      this.mediaRecorder.ondataavailable = (event) => {
        this.audioChunks.push(event.data);
      };
      this.isRecording = true;
      this.startTimer();
    },
    stopRecording() {
      this.mediaRecorder.stop();
      this.mediaRecorder.onstop = async () => {
        const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav' });
        const audioUrl = URL.createObjectURL(audioBlob);
        this.audioUrl = audioUrl;
        this.audioChunks = [];
        this.isRecording = false;
        this.stopTimer();
      };
    },

    handleDragEnter() {
      this.isDragging = true;
    },

    handleDragLeave() {
      this.isDragging = false;
    },

    handleDrop(event) {
      this.isDragging = false;
      const files = event.dataTransfer.files;
      if (files.length) {
        this.audioUrl = URL.createObjectURL(files[0]);

      }
    }
  }
}

</script> 
