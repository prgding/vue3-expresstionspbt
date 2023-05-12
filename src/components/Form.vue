<script setup lang="ts">
import {onMounted, reactive, ref, watch} from 'vue'
import axios from "axios";

let question = ref('　')
let result = ref('　')
let passed = ref('')
let strange = ref('')

const form = reactive({
  input: '',
})

watch(
    () => form.input,
    (val) => {
      if (val === '2') {
        onPass()
      } else if (val === '3') {
        onStrange()
      } else if (val === '4') {
        onReset()
      }
    }
)

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
  if (form.input === '') {
    result.value = '请键入答案或快捷键'
  } else {
    const params = new URLSearchParams();
    params.append('question', `${question.value}`);
    params.append('input', `${form.input}`);
    axios.post('http://127.0.0.1:8080/check', params).then(function (response) {
      if (form.input === response.data) {
        onPass()
        result.value = '正确'
        cleanInput()
      } else {
        result.value = '错误'
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
    if (res.data.indexOf("全部") != 0) {
      passed.value += res.data + '<br>'
      getQuestion()
    } else {
      result.value = "全部提问过了，重新开始吧！"
    }
  }).catch(err => {
    alert("error == " + err)
  })
}

function onStrange() {
  cleanInputAndMsg()
  const params = new URLSearchParams();
  params.append('question', `${question.value}`);
  axios.post('http://localhost:8080/doStrange', params).then(res => {
    if (res.data.indexOf("全部") != 0) {
      strange.value += res.data + '<br>'
      getQuestion()
    } else {
      result.value = "全部提问过了，重新开始吧！"
    }
  }).catch(err => {
    alert("error == " + err)
  })
}

function onReset() {
  if (!confirm("确定要重置吗？")) {
    return
  }
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
  result.value = '　'
}

function resetClean() {
  form.input = ''
  result.value = '　'
  passed.value = ''
  strange.value = ''
}


const cleanMsg = () => {
  result.value = '　'
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
        {{ result }}

        <div id="passed">
          <h2>通过的</h2>
          <el-scrollbar height="50vh" always>
            <p v-for="item in passed.split('<br>').filter(item => item)"
               :key="item" class="scrollbar-demo-item">
              {{ item }}
            </p>
          </el-scrollbar>
        </div>

      </el-form>
    </div>

    <div class="right">
      <h2>不熟悉的</h2>
      <el-scrollbar height="69vh" always>
        <p v-for="item in strange.split('<br>').filter(item => item)"
           :key="item" class="scrollbar-demo-item">
          {{ item }}
        </p>
      </el-scrollbar>
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
  border-right: solid;
}

/* 右侧容器样式 */
.right {
  padding-left: 50px;
  flex-grow: 1; /* 占据父元素剩余的空间 */
}

.scrollbar-demo-item {
  border-radius: 4px;
  text-align: left;
  color: black;
}
</style>