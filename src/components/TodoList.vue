<template>
  <div>
    <div v-for="title in titles" :key="title.id" class="todo-title">
      <div v-if="!title.editing" @dblclick="editTodoTitle(title)" class="todo-title-lable">{{ title.text }}
      </div>
      <input v-else class="todo-title-edit" type="text" v-model="title.text" @blur="doneEditTitle(title)"
        @keyup.enter="doneEditTitle(title)" @keyup.esc="cancelEditTitle(title)" v-focus>
    </div>
    <input type="text" class="todo-input" placeholder="Что нужно сделать?" v-model="newTodo" @keyup.enter="addTodo">
    <draggable v-model="todos" v-bind="dragOptions" @start="drag = true" @end="drag = false">
      <todo-item v-for="(todo, index) in todosFiltered" :key="todo.id" :todo="todo" :index="index"
        :checkAll="anyRemaining" @removedTodo="removeTodo" @finishedEdit="finishedEdit">
      </todo-item>
    </draggable>
    <div class="extra-container">
      <div><label><input type="checkbox" :checked="!anyRemaining" @change="checAllTodos">
          Отметить всё</label></div>
      <div>{{ remaining }} Не выплонено</div>
    </div>
    <div class="extra-container">
      <div>
        <button :class="{ active: filter == 'all' }" @click="filter
          = 'all'">Все</button>
        <button :class="{ active: filter == 'active' }" @click="filter = 'active'">Активные</button>
        <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">Выполненные</button>
        <button @click="exportData">Сохранить проект</button>
        <input type="file" @change="importData">
      </div>
      <div>
        <button v-if="showClearCompletedButton" @click="clearCompleted"> Удалить выполненные </button>
      </div>
    </div>
  </div>
</template>

<script>
import TodoItem from './TodoItem.vue';
import draggable from 'vuedraggable';

export default {
  name: 'todo-list',
  components: {
    TodoItem,
    draggable,
  },
  data() {
    return {
      newTodo: '',
      idForTodo: 3,
      beforeEditCache: '',
      filter: 'all',
      todos: [],
      titles: [
        {
          'id': 0,
          'text': 'Введите название проекта',
          'editing': false,
        }
      ],
      dragOptions: {
        animation: 150,
      },
    }
  },
  computed: {
    remaining() {
      return this.todos.filter(todo => !todo.completed).length
    },
    anyRemaining() {
      return this.remaining !== 0;
    },
    todosFiltered() {
      if (this.todos.length !== 0) {
        if (this.filter == 'all') {
          this.saveToSessionStorage();
          return this.todos;
        } else if (this.filter == 'active') {
          this.saveToSessionStorage();
          return this.todos.filter(todo => !todo.completed)
        } else if (this.filter == 'completed') {
          this.saveToSessionStorage();
          return this.todos.filter(todo => todo.completed)
        }
        return this.todos;
      }
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0
    }
  },

  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim().length == 0) {
        return;
      }

      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        completed: false,
        editing: false,
      })
      this.newTodo = ''
      this.idForTodo++
      this.saveToSessionStorage();
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
      this.saveToSessionStorage();
    },
    checAllTodos() {
      this.todos.forEach((todo) => todo.completed =
        event.target.checked)
      this.saveToSessionStorage();
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed)
      this.saveToSessionStorage();
    },
    finishedEdit(data) {
      this.todos.splice(data.index, 1, data.todo)
      this.saveToSessionStorage();
    },
    editTodoTitle(title) {
      this.beforeEditCache = title.text
      title.editing = true
    },
    doneEditTitle(title) {
      if (title.text != '') {
        document.title = title.text;
        title.editing = false;
        this.saveToSessionStorage();
      } else {
        title.text = 'Введите название проекта';
        document.title = title.text;
        title.editing = false
        this.saveToSessionStorage()
      }
    },
    cancelEditTitle(title) {

      title.text = this.beforeEditCache
      title.editing = false
    },
    saveToSessionStorage() {
      sessionStorage.setItem('todos', JSON.stringify(this.todos));
      sessionStorage.setItem('titles', JSON.stringify(this.titles));
      sessionStorage.setItem('filter', JSON.stringify(this.filter));
    },
    exportData() {
      const projectTitle = this.titles[0].text.trim();
      const exportData = {
        todos: this.todos,
        titles: this.titles,
        filter: this.filter,
      };
      const jsonData = JSON.stringify(exportData);
      const blob = new Blob([jsonData], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      if (projectTitle !== 'Введите название проекта') {
        a.download = `${projectTitle}.json`;
      } else {
        a.download = `Todo.json`;
      }
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    },

    importData(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        const importedData = JSON.parse(e.target.result);
        this.todos = importedData.todos || [];
        this.titles = importedData.titles || [];
        this.saveToSessionStorage(); // Сохраняем данные в sessionStorage
        document.title = this.titles[0].text;
      };
      reader.readAsText(file);
    },

  },
  mounted() {
    const storedTodos = sessionStorage.getItem('todos');
    if (storedTodos) {
      this.todos = JSON.parse(storedTodos);
      this.idForTodo = this.todos.length;

    }
    const storedTitles = sessionStorage.getItem('titles');
    if (storedTitles) {
      this.titles = JSON.parse(storedTitles);
      if (this.titles[0].text == 'Введите название проекта') {
        document.title = "TODO";
      } else {
        document.title = this.titles[0].text;
      }
      const storedFilter = sessionStorage.getItem('filter');
      if (storedFilter) {
        this.filter = JSON.parse(storedFilter);

      }
    }
  },


}
</script>


<style lang="scss">
.todo-input {
  width: 100%;
  padding: 10px 18px;
  font-size: 18px;
  margin-bottom: 16px;
  border-radius: 10px;

  &focus {
    outline: 0;
  }
}

.todo-item {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.todo-title {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;

}

.remove-item {
  cursor: pointer;
  margin-left: 14px;

  &:hover {
    color: black;
  }

}

.todo-item-left {
  display: flex;
  // align-items: center;
}

.todo-title-label {
  width: 100%;
  color: #2c3e50;
  padding: 10px 18px;
  font-size: 18px;
  margin-bottom: 16px;
  border-radius: 10px;
  font-family: Arial, Helvetica, sans-serif;

  &focus {
    outline: 0;
  }
}

.todo-item-label {
  font-size: 24px;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  margin-left: 12px;
  padding: 10px;
  border: 1px solid #ccc;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 10px;
}

.todo-title-edit {
  font-size: 24px;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 10px;

  &:focus {
    outline: none;
  }
}

.todo-item-edit {
  font-size: 24px;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  margin-left: 12px;
  padding: 10px;
  border: 1px solid #ccc;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 10px;

  &:focus {
    outline: none;
  }
}

.extra-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 16px;
  border-top: 1px solid lightgrey;
  padding-top: 14px;
  margin-bottom: 14px;
}

button {
  font-size: 14px;
  background: white;
  appearance: none;

  &:hover {
    background: lightgreen;
  }

  &:focus {
    outline: none;
  }
}

.active {
  background: lightgreen;
}

.todo-container {
  display: flex;
  flex-direction: column;
}

.todo-description {
  width: 550px;
  word-wrap: break-word;
}

.todo-item-description {
  font-size: 16px;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  margin-left: 12px;
  padding: 10px;
  border: 1px solid #ccc;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 10px;
  border-top: white;
}

.todo-item-description-input {
  width: 100%;
  height: 100%;
  padding: 10px;
  margin-left: 12px;
  color: #2c3e50;
  border: 1px solid #ccc;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
  resize: none;
}

.completed {
  text-decoration: line-through;
  color: grey;
}
</style>
