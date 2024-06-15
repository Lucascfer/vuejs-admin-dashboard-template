<template>
    <div class="drag-drop-area border border-dashed border-gray-400 p-10 w-full flex align-center flex-col justify-center items-center space-x-4  max-w-9xl mx-auto p-4"
        @dragover.prevent @dragenter.prevent="handleDragEnter" @dragleave="handleDragLeave" @drop="handleDrop">
        <p v-if="!isDragging">Drag or drop file here to update</p>
        <p v-else>Drop the file now...</p>
    </div>
</template>

<script>
import { ref } from 'vue'
export default {
    name: 'DragOrDropFile',
    setup() {
        return {
            isDragging: ref(false),
        }
    },
    methods: {
        emitAudioUrl(url) {
            this.$emit('audio-recorded', url);
        },
        handleDragEnter() {
            this.isDragging = true;
        },

        handleDragLeave() {
            this.isDragging = false;
        },

        handleDrop(event) {
            this.handleDragLeave()
            const files = event.dataTransfer.files;
            if (files.length) {
                this.emitAudioUrl(URL.createObjectURL(files[0]));
            }
        }
    }
}    
</script>