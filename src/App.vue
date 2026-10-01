<script setup>
import { ref } from 'vue';
import DateForm from './components/form.vue';

const started = ref(false);
const transitioning = ref(false);

const button = ref(null);
const position = ref(null);

function moveButton() {
  const { width, height } = button.value.getBoundingClientRect()
  const padding = 20;

  document.querySelector('#accept').classList.add('full')

  const maxX = Math.max(padding, window.innerWidth - width - padding)
  const maxY = Math.max(padding, window.innerHeight - height - padding)

  position.value = {
    left: `${padding + Math.random() * (maxX - padding)}px`,
    top: `${padding + Math.random() * (maxY - padding)}px`,
  }
}

</script>

<template>
  <div class="main-part">
    <div class="card-scene" :inert="transitioning">
      <Transition name="card-flip" mode="out-in" @before-leave="transitioning = true"
        @after-enter="transitioning = false">
        <div v-if="!started" key="invitation" class="flirt-card">
          <h1 class="flirt-card__title">В честь нашей первой годовщины наших отношений</h1>
          <p class="flirt-card__subtitle">Дорогая, Софья, хочу вас пригласить на наше свидание и хочу предоставить выбор
            вам, куда мы пойдем.</p>
          <img src="./images/kitten-transparent.png" class="flirt-card__image" alt="Котик" />
          <p class="flirt-card__cta">Вы пойдете со мною на свидание?))))))</p>
          <div class="flirt-card__footer">
            <button class="flirt-card__button flirt-card__button--accept" type="button" id="accept"
              @click="started = true">ДА!</button>
            <button class="flirt-card__button flirt-card__button--decline" type="button" ref="button" id="decline"
              :style="position ? { position: 'fixed', ...position } : {}" @click="moveButton">нет 🙃</button>
          </div>
        </div>
        <DateForm v-else key="questions" />
      </Transition>
    </div>
  </div>
</template>

<style scoped></style>
