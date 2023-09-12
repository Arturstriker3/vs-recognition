<template>
  <div class="app">
    <button :class="['mic', isRecording ? 'recording' : '']" @click="toggleMic">
      <i :class="['fa', 'fa-microphone']" :style="{ color: isRecording ? 'red' : '' }"></i> {{ isRecording ? 'Stop' : 'Start' }}
    </button>
    <div class="transcript" v-text="transcript"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const transcript = ref('')
const isRecording = ref(false)

const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition
const sr = new Recognition()

const toggleMic = () => { // Exponha a função como toggleMic
  if (isRecording.value) {
    sr.stop()
  } else {
    sr.start()
  }
}

onMounted(() => {
  sr.continuous = true
  sr.interimResults = true

  sr.onstart = () => {
    console.log('SR Started')
    isRecording.value = true
  }

  sr.onend = () => {
    console.log('SR Stopped')
    isRecording.value = false
  }

  sr.onresult = (evt) => {
    for (let i = 0; i < evt.results.length; i++) {
      const result = evt.results[i]

      if (result.isFinal) CheckForCommand(result)
    }

    const t = Array.from(evt.results)
      .map(result => result[0])
      .map(result => result.transcript)
      .join('')

    transcript.value = t
  }
})

const CheckForCommand = (result) => {
  const t = result[0].transcript;
  if (t.includes('parar a gravação')||
      t.includes('parar gravação')
  ) {
    sr.stop()
  }
}
</script>

<style scoped>

  .app{
    display: flex;
    flex-direction: column;
    justify-content: center; /* Centraliza horizontalmente */
    align-items: center; /* Centraliza verticalmente */
    height: 100vh; /* Define a altura para ocupar a altura da janela */
  }

  .transcript{
    margin-top: 40px;
    text-align: justify;
    font-size: 20px;
  }

  .mic{
  font-family: 'fontello';
  font-style: normal;
  font-size: 3em;
  speak: none;
  width: 150px; /* Defina o tamanho desejado para criar um botão redondo */
  height: 150px; /* Defina o tamanho desejado para criar um botão redondo */
  border-radius: 50%; /* Torna o botão redondo definindo a borda como 50% do tamanho */
  border: 2px solid black; /* Adicione uma borda preta de 2px */
  }

  .mic:hover{
    filter: drop-shadow(0 0 0.5em #FFF);
    background-color: #35495e;
    opacity: 0.85;
    cursor: pointer;
  }
</style>
