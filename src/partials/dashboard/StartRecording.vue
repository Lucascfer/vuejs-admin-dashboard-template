<template>
    <div
        class="border border-dashed border-gray-400 p-10 w-full flex align-center flex-col justify-center items-center max-w-9xl mx-auto p-4">
        <button @click="toggleRecording"
            :class="isRecording ? 'bg-red-500 text-white' : ' hover:bg-green-700 hover:text-white'"
            class="text-black py-2 px-4 rounded">
            {{ isRecording ? 'Stop Recording' : 'Start Recording' }}
        </button>
        <div class="text-sm text-gray-500 text-center mt-2">
            {{ formatTime(recordingTime) }}
        </div>
    </div>
</template>

<script>
import { ref } from 'vue'
export default {
    name: 'StartRecording',
    setup() {
        return {
            mediaRecorder: ref(null),
            audioChunks: [],
            isRecording: ref(false),
            recordingTime: ref(0),
        }
    },
    methods: {
        emitAudioUrl(url) {
            this.$emit('audio-recorded', url, this.recordingTime);
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
                const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav'});
                const audioUrlValue = URL.createObjectURL(audioBlob);
                this.audioChunks = [];
                this.isRecording = false;
                this.stopTimer();
                this.emitAudioUrl(audioUrlValue);
            };
        },
    },

}    
</script>