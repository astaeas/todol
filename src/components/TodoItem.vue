<template>
    <div class="todo-item">
        <div class="todo-item-left">
            <input type="checkbox" v-model="completed" @change="doneEdit">
            <div class="todo-container">
                <div class="todo-item-title">
                    <div v-if="!editing" @dblclick="editTodo" class="todo-item-label" :class="{ completed: completed }">
                        {{ title }}</div>
                    <input v-else class="todo-item-edit" type="text" v-model="title" @blur="doneEdit"
                        @keyup.enter="doneEdit" @keyup.esc="cancelEdit" v-focus>
                </div>
                <div class=todo-description>
                    <div v-if="!editingDescription" @dblclick="editTodoDescription" class="todo-item-description"
                        :class="{ completed: completed }">
                        {{ description }}</div>
                    <textarea v-else class="todo-item-description-input" type="text" v-model="description"
                        @blur="doneEditDescription" @keyup.enter="doneEditDescription" @keyup.esc="cancelEditDescription"
                        v-focus>
                    </textarea>
                </div>
            </div>
        </div>
        <div class="remove-item" @click="removeTodo(index)">
            &times;
        </div>
    </div>
</template>

<script>

export default {
    name: 'todo-item',
    props: {
        todo: {
            type: Object,
            required: true,
        },
        index: {
            type: Number,
            required: true,
        },
        checkAll: {
            type: Boolean,
            required: true,
        }
    },
    data() {
        return {
            'id': this.todo.id,
            'title': this.todo.title,
            'completed': this.todo.completed,
            'editing': this.todo.editing,
            'description': this.todo.description,
            'editingDescription': this.todo.editingDescription,
            'beforeEditCache': '',
            'beforeEditCacheDesc': '',
        }
    },
    watch: {
        checkAll() {
            if (this.checkAll) {
                this.completed = this.todo.completed;
            } else {
                this.completed = this.todo.completed;
            }
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
        removeTodo(index) {
            this.$emit('removedTodo', index);
            console.log(index);
        },
        editTodo() {
            this.beforeEditCache = this.title;
            this.editing = true;
        },
        doneEdit() {
            if (this.title.trim() == '') {
                this.title = this.beforeEditCache;
            }
            this.editing = false;
            this.$emit('finishedEdit', {
                'index': this.index,
                'todo': {
                    'id': this.id,
                    'title': this.title,
                    'completed': this.completed,
                    'editing': this.editing,
                }
            });
        },
        editTodoDescription() {
            this.beforeEditCacheDesc = this.description;
            this.editingDescription = true;
        },
        doneEditDescription() {
            if (this.description.trim() == '') {
                this.description = this.beforeEditCacheDesc;
            }
            this.editingDescription = false;
            this.$emit('finishedEdit', {
                'index': this.index,
                'todo': {
                    'editingDescription': this.editingDescription,
                    'description': this.description,
                    'id': this.id,
                    'title': this.title,
                    'completed': this.completed,
                    'editing': this.editing,
                }
            });
        },
        cancelEdit() {
            this.title = this.beforeEditCache
            this.editing = false
        },
        cancelEditDescription() {
            this.description = this.beforeEditCacheDesc
            this.editingDescription = false
        },
    }
}
</script>