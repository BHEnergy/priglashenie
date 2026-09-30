<script setup>
import { computed, nextTick, onMounted, ref } from 'vue'

// Заготовки: тексты и варианты можно заменить здесь.
const questions = [
  { title: 'В какой ресторан мы пойдем?', options: ['Пипяо', 'Чико', 'Хинкали хаус', 'Твой вариант'] },
  { title: 'Во сколько мы с тобою пойдем?', options: ['14:00', '16:00', '18:00', '20:00'] },
  { title: 'Чем будем заниматься после?', options: ['Ебланить', 'Смотреть фильм на твой выбор', 'Прогуляемся', 'Ничем'] },
  { title: 'Что думаешь?', options: ['Согласна', 'Просто "Да"', '!Не согласна', 'Ебать ты...'] }
]
const currentStep = ref(0)
const answers = ref(questions.map(() => ''))
const submitted = ref(false)
const sending = ref(false)
const submitError = ref('')
const transitioning = ref(false)
const heading = ref(null)
const canContinue = computed(() => answers.value[currentStep.value].length > 0)
const isLastStep = computed(() => currentStep.value === questions.length - 1)

function focusHeading() {
  heading.value?.focus({ preventScroll: true })
}
onMounted(() => nextTick(focusHeading))

function finishTransition() {
  transitioning.value = false
  focusHeading()
}

function nextStep() {
  if (!transitioning.value && canContinue.value && !isLastStep.value) currentStep.value++
}

async function handleSubmit() {
  if (transitioning.value || submitted.value || sending.value) return
  if (!isLastStep.value) return nextStep()
  if (answers.value.some((answer) => answer.length === 0)) return
  sending.value = true
  submitError.value = ''
  const controller = new AbortController()
  const timeout = setTimeout(() => controller.abort(), 20000)
  try {
    // Берём ответы всех шагов из состояния: предыдущих полей уже нет в DOM.
    const response = await fetch('https://formspree.io/f/mjykzjyy', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', Accept: 'application/json' },
      body: JSON.stringify(Object.fromEntries(
        questions.map((question, index) => [question.title, answers.value[index]]),
      )),
      signal: controller.signal,
    })
    if (!response.ok) throw new Error('Submission rejected')
    submitted.value = true
  } catch (error) {
    submitError.value = error.name === 'AbortError'
      ? 'Не удалось дождаться подтверждения. Ответы сохранены — попробуй отправить ещё раз.'
      : 'Не удалось отправить ответы. Проверь подключение и попробуй ещё раз.'
  } finally {
    clearTimeout(timeout)
    sending.value = false
  }
}
</script>

<template>
  <form class="date-form" :inert="transitioning" :aria-busy="sending" @submit.prevent="handleSubmit">
    <Transition name="card-flip" mode="out-in" @before-leave="transitioning = true" @after-enter="finishTransition">
      <section v-if="submitted" key="result" class="flirt-card question-card" aria-labelledby="result-title">
        <p class="step-caption">Наш план свидания</p>
        <h2 id="result-title" ref="heading" class="flirt-card__title" tabindex="-1">Ответы отправлены ♡</h2>
        <dl class="answer-summary">
          <div v-for="(question, index) in questions" :key="question.title">
            <dt>{{ question.title }}</dt>
            <dd>{{ answers[index] }}</dd>
          </div>
        </dl>
        <p class="form-hint">Прекрасно) Я получил твои ответы, поэтому все организую</p>
      </section>

      <section v-else :key="currentStep" class="flirt-card question-card" aria-labelledby="question-title">
        <div class="step-progress" aria-label="Этапы формы">
          <span v-for="(_, index) in questions" :key="index" :class="{ active: index <= currentStep }"
            :aria-current="index === currentStep ? 'step' : undefined"></span>
        </div>
        <p class="step-caption">Шаг {{ currentStep + 1 }} из {{ questions.length }}</p>
        <h2 id="question-title" ref="heading" class="flirt-card__title" tabindex="-1">{{ questions[currentStep].title }}
        </h2>
        <p id="selection-hint" class="form-hint">Выбери один вариант</p>
        <fieldset class="question-options" aria-describedby="selection-hint" :disabled="sending">
          <legend class="visually-hidden">{{ questions[currentStep].title }}</legend>
          <label v-for="(option, index) in questions[currentStep].options" :key="option" class="question-option"
            :class="{ selected: answers[currentStep] === option }">
            <input v-model="answers[currentStep]" type="radio" :name="`question-${currentStep}`" :value="option" />
            <span class="option-number">0{{ index + 1 }}</span>
            <span>{{ option }}</span>
          </label>
        </fieldset>
        <div class="question-footer">
          <button v-if="currentStep > 0" type="button" class="form-navigation" :disabled="sending"
            @click="currentStep--; submitError = ''">← Назад</button>
          <span v-else></span>
          <Transition name="next-arrow">
            <button v-if="canContinue" :key="currentStep" :type="isLastStep ? 'submit' : 'button'"
              class="form-navigation form-navigation--next" :disabled="sending" @click="!isLastStep && nextStep()">
              {{ sending ? 'Отправляем…' : isLastStep ? (submitError ? 'Повторить отправку' : 'Отправить') : 'Далее' }}
              <span aria-hidden="true">→</span>
            </button>
          </Transition>
        </div>
        <p v-if="submitError" class="form-hint form-error" role="alert">{{ submitError }}</p>
        <span class="visually-hidden" role="status">{{ sending ? 'Отправляем ответы' : '' }}</span>
      </section>
    </Transition>
  </form>
</template>
