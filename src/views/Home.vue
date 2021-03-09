<template>
  <div class="home">
    <el-row>
      <el-col :span="2">
      </el-col>
      <el-col :span="20">
        <el-row>
          <p class="label">Number: {{qNumber}}</p>
          <!-- <el-form inline size="mini" class="number-form">
            <el-form-item label="Number:" size="mini">
              <el-input type="number" disabled v-model="qNumber" size="mini"/>
            </el-form-item>
          </el-form> -->
        </el-row>
        <el-row>
          <p class="label">QuestionText</p>
          <el-input
            type="textarea"
            :rows="14"
            placeholder="Please input"
            v-model="questionText"
          />
        </el-row>
        <el-row type="flex" align="bottom">
          <el-col :span="4">
            <p class="label">Answer</p>
          </el-col>
          <el-col :span="4">
            <el-checkbox v-model="expOn">Explanation</el-checkbox>
          </el-col>
        </el-row>
        <el-row v-show="(questionText)">
          <el-checkbox-group v-model="answer" size="large">
            <el-checkbox-button v-for="choice in choicesKey" :label="choice" :key="choice">{{choice}}</el-checkbox-button>
          </el-checkbox-group>
          <el-input
            type="textarea"
            :rows="4"
            v-model="explanation"
            v-show="(expOn)"
          />
        </el-row>
        <el-row style="margin-top: 20px">
          <el-button type="primary" :disabled="answer.length < 1" @click="saveQuestion">Next</el-button>
        </el-row>
      </el-col>
      <el-col :span="2">
      </el-col>
    </el-row>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, ref, onMounted, toRaw, Ref, setTransitionHooks } from 'vue';
import { setState, getState, deleteState } from '@/storage'

interface Quiz {
  qNumber: number;
  question: string;
  choices: {};
  answer: string[];
  explanation?: string;
}

export default defineComponent({
  name: 'Home',
  setup(){
    // ref
    const quizList: Ref<Quiz[]> = ref([])
    const questionText = ref('')
    const expOn = ref(false)
    const explanation = ref('')
    const answer = ref([])

    // computed
    const qNumber =  computed((): number => quizList.value?quizList.value.length+1:1)
    const question = computed((): string => questionText.value.split("\nA. ")[0])    
    const choicesDict = computed(() => {
      return questionText.value.slice(question.value.length+1).split("\n").reduce((accum, value)=>{
        if (['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I'].includes(value[0])){
          return Object.assign(accum, {[value[0]]: value.slice(3)})
        } else {
          return accum
        }
      },{})
    })
    const choicesKey = computed(()=>Object.keys(choicesDict.value))

    // methods
    const initialize = () => {
      questionText.value = ''
      expOn.value = false
      explanation.value = ''
      answer.value = []
    }
    const saveQuestion = () => {
      const quiz = {
        qNumber: qNumber.value,
        question: question.value,
        choices: toRaw(choicesDict.value),
        answer: toRaw(answer.value),
        explanation: explanation.value
      }
      console.log(quiz)
      quizList.value.push(quiz)
      setState('quiz', toRaw(quizList.value))
      initialize()
    }
    const loadData = async ()=>{
      const savedData = await getState('quiz')
      quizList.value = savedData?savedData:[]
    }

    // lifecycle hooks
    onMounted(()=>{
      loadData()
    })

    window.myAPI.openData(async (examData: string)=>{
      console.log(examData)
      await setState('quiz', JSON.parse(examData))
      loadData()
    })
    window.myAPI.saveData((filepath: string)=>{
      window.myAPI.sendSaveData(filepath, toRaw(quizList.value))
    })
    window.myAPI.deleteData(()=>{
      deleteState()
      loadData()
    })

    return {
      qNumber,
      questionText,
      choicesKey,
      expOn,
      explanation,
      answer,
      saveQuestion
    }
  }
});
</script>

<style>
p.label {
  margin-bottom: 0;
}

.number-form > div.el-form-item {
  margin-bottom: 0;
}

</style>