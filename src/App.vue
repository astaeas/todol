<template>
  <div id="app" class="container">
    <todo-list> </todo-list>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import TodoList from './components/TodoList.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    TodoList,
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

#app{
  font-size: 24px;
}

.container {
  max-width: 600px;
  margin: 0 auto;
}
</style>
