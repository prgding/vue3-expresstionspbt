<template>
  <h1>App</h1>
  <Form/>
</template>

<script lang="ts">
import {Options, Vue} from 'vue-class-component';
import Form from './components/Form.vue';

@Options({
  components: {
    Form,
  },
})
export default class App extends Vue {
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 50px;
}
</style>