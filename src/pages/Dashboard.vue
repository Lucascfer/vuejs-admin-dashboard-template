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

          <!-- Create New -->
          <div class="flex justify-end">
            <label class="btn bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 cursor-pointer flex"
              :class="createAudio ? 'bg-red-500 hover:bg-red-600' : 'bg-indigo-500 hover:bg-indigo-600'">
              {{ createAudio ? 'Cancel' : 'Create New' }}
              <button @click="createAudio = !createAudio" class="hidden" />
            </label>
          </div>

          <!-- Actions -->
          <div class="sm:flex sm:justify-between sm:items-center mb-8">
            <div class="grid grid-flow-col sm:auto-cols-max justify-start sm:justify-end gap-2">
              <h2 class="text-2x2 font-semibold text-slate-800 px-4 py-2 inline-block cursor-pointer"
                @click="createAudio = false">Your recordings</h2>

              <!-- Add script button -->
              <label class="btn bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 cursor-pointer inline-block">
                Add script
                <input type="file" @change="handleFileUpload" class="hidden" />
              </label>
            </div>
          </div>

          <!-- Recordings -->
          <div v-if="createAudio" class="m-4">
            <StartRecording @audio-recorded="handleAudioRecorded" />
            <span
              class="text-black font-semibold text-xl m-2 text-center block before:content-['-'] after:content-['-'] before:mr-2 after:ml-2">
              Or
            </span>
            <DragOrDropFile @audio-recorded="handleAudioRecorded" />
          </div>

          <!-- Audios recorded -->
          <div v-if="!createAudio">
            <div v-if="recordings.length === 0" class="text-center text-xl">
              You have no recordings yet <br>
              <span class="text-black font-semibold">Start by making a recording.</span>
            </div>

            <div v-if="recordings.length > 0" class="mt-4 space-y-4">
              <div v-for="(recording, index) in recordings" :key="index"
                class="flex items-center bg-blue-500 text-white p-4 rounded-lg shadow-lg">
                <div class="flex-grow">
                  <div class="flex items-center space-x-4">
                    <audio controls ref="audio" :src="recording.url" @play="setCurrentPlaying(index), actuallyPlaying = index"
                      @pause="setCurrentPlaying(null), playing = !playing">
                    </audio>
                    <div class="flex justify-evenly w-full">
                      <div class="text-lg font-semibold cursor-pointer w-1/2 px-4" @dblclick="editName(index)">{{ recording.name }}
                      </div>
                      <div class="mr-8 flex flex-row space-x-2 w-full justify-end items-center">
                        <div class="text-sm">{{ recording.date }}</div>
                        <div class="text-sm">{{ recording.length !== Infinity ? formatTime(recording.length) : '0:00' }}</div>
                      </div>
                    </div>
                  </div>
                </div>
                <button @click="deleteRecording(index)" class="text-white-400 hover:text-red-900 material-icons"> delete
                </button>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import Sidebar from '../partials/Sidebar.vue';
import Header from '../partials/Header.vue';
import WelcomeBanner from '../partials/dashboard/WelcomeBanner.vue';
import StartRecording from '../partials/dashboard/StartRecording.vue';
import DragOrDropFile from '../partials/dashboard/DragOrDropFile.vue';

export default {
  name: 'Dashboard',
  components: {
    Sidebar,
    Header,
    WelcomeBanner,
    StartRecording,
    DragOrDropFile,

  },
  setup() {
    const sidebarOpen = ref(false);
    const createAudio = ref(true);
    const recordings = ref([]); //backend
    const currentPlaying = ref(null);

    const togglePlayPause = (index) => {
      const audioElement = document.querySelectorAll('audio')[index];
      if (currentPlaying.value === index) {
        audioElement.pause();
      } else {
        audioElement.play();
      }
    };

    const setCurrentPlaying = (index) => {
      currentPlaying.value = index;
    };

    const handleAudioRecorded = (url, recordingTime) => {
      const audio = new Audio(url);
      audio.addEventListener('loadedmetadata', () => {
        const length = audio.duration;
        recordings.value.push({
          url,
          name: `Recording ${recordings.value.length + 1}`,
          date: new Date().toLocaleString(),
          length: recordingTime ? recordingTime : length,
        });
        createAudio.value = false;
      });
    };

    const handleFileUpload = (event) => {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          const url = reader.result;
          handleAudioRecorded(url);
        };
        reader.readAsDataURL(file);
      }
    };

    const deleteRecording = (index) => {
      recordings.value.splice(index, 1);
    };

    const editName = (index) => {
      const newName = prompt('Enter new name:', recordings.value[index].name);
      if (newName) {
        recordings.value[index].name = newName;
      }
    };

    const formatTime = (seconds) => {
      const minutes = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${minutes}:${secs < 10 ? '0' : ''}${secs}`;
    };

    return {
      sidebarOpen,
      createAudio,
      recordings,
      handleAudioRecorded,
      handleFileUpload,
      deleteRecording,
      editName,
      formatTime,
      togglePlayPause,
      setCurrentPlaying,
      playing: ref(false),
      actuallyPlaying: ref(0),
    };
  },
};
</script>
