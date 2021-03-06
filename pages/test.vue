<template>
  <v-container v-resize="onResize">
    <Language v-if="!form.language" />
    <div v-else>
      <v-row>
        <v-col cols="10" class="pa-0">
          <TestLanguageSwitcher />
        </v-col>
        <v-col>
          <Timer />
        </v-col>
      </v-row>
      <v-progress-linear
        :value="GET_PROGRESS"
        height="25"
        reactive
        rounded
        background-opacity="0.1"
        class="mb-3"
        :style="GET_PROGRESS > 49 && { color: 'white' }"
      >
        {{ GET_PROGRESS }}%
      </v-progress-linear>
      <div v-for="question in GET_CURRENT_QUESTIONS" :key="question.id">
        <h2>{{ question.text }}</h2>
        <v-radio-group @change="answer => SET_ANSWER({ id: question.id, answer } )" :value="GET_CURRENT_ANSWER(question.id)">
          <v-radio
            v-for="(choice, i) in question.choices"
            :name="question.id"
            :key="i"
            :value="choice.score"
            :off-icon="mdiRadioboxBlank"
            :on-icon="mdiRadioboxMarked"
            color="secondary"
            :label="choice.text"
            ></v-radio>
        </v-radio-group>
      </div>

      <v-row class="justify-center pt-4">
        <div v-if="!test.done">
          <v-btn large color="primary" :disabled="BACK_BUTTON_STATE" class="mr-2" @click="PREVIOUS_QUESTIONS">
            {{ $t("test.back") }}
          </v-btn>

          <v-btn large color="primary" :disabled="NEXT_BUTTON_STATE" @click="NEXT_QUESTIONS">
            {{ $t("test.next") }}
          </v-btn>
          <v-btn v-if="development" large color="primary" class="ml-2" @click="SKIP_QUESTIONS">
            dev: skip to end
          </v-btn>
        </div>

        <div v-else-if="loading" class="text-center">
          <v-progress-circular
            indeterminate
            color="secondary"
            size="128"
          ></v-progress-circular>
      </div>

        <div v-else>
          <v-btn large color="secondary" @click="SUBMIT_TEST">
            {{ $t('test.seeResults') }}
          </v-btn>
        </div>

      </v-row>

    </div>
  </v-container>
</template>

<script>
import { mdiRadioboxMarked, mdiRadioboxBlank } from '@mdi/js'
import { mapMutations, mapState, mapGetters, mapActions } from 'vuex'
import { sleep } from '../lib/helpers'

export default {
  name: 'test',
  head: () => ({
    title: 'The test'
  }),
  mounted () {
    this.onResize(),
    this.$amplitude.getInstance().logEvent('b5.test', { part: 'start' });
  },
  data: () => ({
    mdiRadioboxBlank,
    mdiRadioboxMarked
  }),
  methods: {
    ...mapMutations(['SET_INVENTORY', 'SET_ANSWER', 'NEXT_QUESTIONS', 'PREVIOUS_QUESTIONS', 'SET_ITEMS_PER_PAGE', 'SKIP_QUESTIONS']),
    ...mapActions(['SUBMIT_TEST']),
    onResize () {
      window.innerWidth < 600
        ? this.SET_ITEMS_PER_PAGE(1)
        : this.SET_ITEMS_PER_PAGE(3)
    }
  },
  computed: {
    ...mapState(['test', 'development', 'form', 'loading']),
    ...mapGetters(['GET_CURRENT_QUESTIONS', 'GET_PROGRESS', 'NEXT_BUTTON_STATE', 'BACK_BUTTON_STATE', 'GET_CURRENT_ANSWER'])
  },
  created () {
    this.SET_INVENTORY()
  },
  components: {
    Timer: () => import('../components/Timer'),
    Language: () => import('../components/form/Language'),
    TestLanguageSwitcher: () => import ('../components/TestLanguageSwitcher')
  },
  watch: {
    'test.done': async function (testDone) {
      if (testDone) {
        this.$confetti.start({ particlesPerFrame: 0.25, particles: [{ type: 'heart' }] })
        await sleep(4000)
        this.$confetti.stop()
      }
    }
  }
}
</script>
