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
          <div class="flex justify-end">
            <label class="btn bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 cursor-pointer flex"
              :class="createAudio ? 'bg-red-500 hover:bg-red-600' : 'bg-indigo-500 hover:bg-indigo-600'">
              {{ createAudio ? 'Cancel' : 'Create New' }}
              <button @click="createAudio = !createAudio" class="hidden" />
            </label>
          </div>
          <!-- Dashboard actions -->
          <div class="sm:flex sm:justify-between sm:items-center mb-8">
            <!-- Right: Actions -->
            <div class="grid grid-flow-col sm:auto-cols-max justify-start sm:justify-end gap-2">
              <h2 class="text-2x2 font-semibold text-slate-800 px-4 py-2 inline-block">Your recordings</h2>

              <!-- Add view button -->
              <label class="btn bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 cursor-pointer inline-block">
                Add script
                <input type="file" @change="handleFileUpload" class="hidden" />
              </label>
            </div>
          </div>

          <div v-if="createAudio" class="m-4">
            <StartRecording @audio-recorded="handleAudioRecorded" />

            <span
              class="text-black font-semibold text-xl m-2 text-center block before:content-['-'] after:content-['-'] before:mr-2 after:ml-2">
              Or
            </span>

            <DragOrDropFile @audio-recorded="handleAudioRecorded" />
          </div>

          <div v-if="audioUrl.length < 1" class="text-center text-xl">
            You have no recordings yet <br>
            <span class="text-black font-semibold">Start by making a recording.</span>
          </div>

          <div v-if="audioUrl.length > 0" class="mt-4">
            <audio v-for="(url, index) in audioUrl" :key="index" :src="url" controls class="w-full mb-4"></audio>
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
import StartRecording from '../partials/dashboard/StartRecording.vue'
import DragOrDropFile from '../partials/dashboard/DragOrDropFile.vue'

export default {
  name: 'Dashboard',
  components: {
    Sidebar,
    Header,
    WelcomeBanner,
    StartRecording,
    DragOrDropFile
  },

  setup() {
    const sidebarOpen = ref(false)
    return {
      sidebarOpen,
      audioUrl: ref([]),
      createAudio: ref(true),
    }
  },

  methods: {
    handleAudioRecorded(url) {
      this.audioUrl.push(url);
      this.createAudio = false;
    },

    handleFileUpload() {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          this.audioUrl.push(reader.result);
        };
        reader.readAsDataURL(file);
      }
    },
  }
}

</script> 
