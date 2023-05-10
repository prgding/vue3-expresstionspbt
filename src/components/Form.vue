<script setup lang="ts">
import {onMounted, reactive, ref} from 'vue'
import axios from "axios";

let question = ref('　')
let result = ref('')
let passed = ref('')
let strange = ref('')

const form = reactive({
  input: '',
})

onMounted(() => {
  getQuestion()
})

function getQuestion() {
  axios.get('http://localhost:8080/getQuestion').then(res => {
    question.value = res.data
  }).catch(error => {
    alert("error == " + error)
  })
}

const onSubmit = () => {
  if (form.input === '2') {
    onPass()
  } else if (form.input === '3') {
    onStrange()
  } else if (form.input === '4') {
    onReset()
  } else if (form.input === '') {
    result.value = '请键入答案或快捷键'
  } else {
    const params = new URLSearchParams();
    params.append('question', `${question.value}`);
    params.append('input', `${form.input}`);
    axios.post('http://127.0.0.1:8080/check', params).then(function (response) {
      result.value = `${response.data}`;
      if (result.value === '正确') {
        getQuestion()
        cleanInput()
      }
    }).catch(function (error) {
      alert("error == " + error);
    });
  }
}

function onPass() {
  cleanInputAndMsg()
  const params = new URLSearchParams();
  params.append('question', `${question.value}`);
  axios.post('http://localhost:8080/doPass', params).then(res => {
    passed.value += res.data + '<br>'
    getQuestion()
  }).catch(err => {
    alert("error == " + err)
  })
}

function onStrange() {
  cleanInputAndMsg()
  const params = new URLSearchParams();
  params.append('question', `${question.value}`);
  axios.post('http://localhost:8080/doStrange', params).then(res => {
    strange.value += res.data + '<br>'
    getQuestion()
  }).catch(err => {
    alert("error == " + err)
  })
}

function onReset() {
  cleanInputAndMsg()
  axios.get('http://localhost:8080/doReset').then(res => {
    getQuestion()
    resetClean()
    result.value = res.data
  }).catch(err => {
    alert("error == " + err)
  })
}

const cleanInput = () => {
  form.input = ''
}

const cleanInputAndMsg = () => {
  form.input = ''
  result.value = ''
}

function resetClean() {
  form.input = ''
  result.value = ''
  passed.value = ''
  strange.value = ''
}


const cleanMsg = () => {
  result.value = ''
}


</script>

<template>
  <div class="container">
    <div class="left">
      <el-form
          @submit.prevent="onSubmit"
          class="el-form"
          :label-position="'top'"
          label-width="100px"
          :model="form"
      >

        <el-form-item :label=question>
          <el-input @keydown="cleanMsg" placeholder="快捷键：2: 通过, 3: 不熟悉, 4: 重置提问" v-model="form.input"/>
        </el-form-item>

        <el-row class="mb-4">
          <el-button type="primary" @click="onSubmit">检查</el-button>
          <el-button type="success" @click="onPass">通过</el-button>
          <el-button type="info" @click="onStrange">不熟悉</el-button>
          <el-button type="danger" @click="onReset">重置提问</el-button>
        </el-row>
        <br>
        {{ result }}<br>

        <div id="passed">
          <h2>通过的</h2>
          <span v-html="passed"></span>
        </div>

      </el-form>
    </div>

    <div class="right">
      <h2>陌生的</h2>
      <span v-html="strange"></span>
    </div>
  </div>

</template>

<style scoped>
.el-form {
  margin: 0 50px;
  max-width: 80%;
}

/* 容器样式 */
.container {
  display: flex;
}

/* 左侧容器样式 */
.left {
  width: 50%;
  border-right: solid black;
}

/* 右侧容器样式 */
.right {
  width: 50%;
}
</style>