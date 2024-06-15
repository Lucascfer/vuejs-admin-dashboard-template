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

            <div class="p-4">
              <button @click="toggleRecording" :class="isRecording ? 'bg-red-500' : 'bg-green-500'"
                class="text-white py-2 px-4 rounded">
                {{ isRecording ? 'Stop Recording' : 'Start Recording' }}
              </button>
            </div>
            
          </div>
          <audio v-if="audioUrl" :src="audioUrl" controls class="mt-4 w-full"></audio>

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
      isRecording: ref(false)
    }
  },

  methods: {
    toggleRecording() {
      this.isRecording ? this.stopRecording() : this.startRecording();
    },
    startRecording() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({ audio: true })
          .then(stream => {
            this.mediaRecorder = new MediaRecorder(stream);
            this.mediaRecorder.ondataavailable = event => {
              this.audioChunks.push(event.data);
            };
            this.mediaRecorder.onstop = () => {
              const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav' });
              this.audioUrl = URL.createObjectURL(audioBlob);
              this.audioChunks = [];
              // upload 'audioUrl'to backend
            };
            this.mediaRecorder.start();
            this.isRecording = true;
          })
          .catch(error => {
            console.error('Error accessing microphone:', error);
          });
      } else {
        console.error('The browser does not support audio recording.');
      }
    },
    stopRecording() {
      if (this.mediaRecorder) {
        this.mediaRecorder.stop();
        this.isRecording = false;
      }
    }

  }
}

</script> 
