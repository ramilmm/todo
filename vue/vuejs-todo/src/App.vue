<template>
  <div class="container">
      <div class="row">
          <div class="col-xs-12 main-content">
              <h1>Todo List</h1>

              <todo-form></todo-form>

              <hr />

              <h1>All Tasks</h1>
              <div id="task-list">

                <todos :tasks="tasks | filterTask filter"></todos>

              </div>
              <button @click="filterTask" v-if="tasks.length">All</button>
              <button @click="filterTask" v-if="tasks.length">Active</button>
              <button @click="filterTask" v-if="tasks.length">Completed</button>
          </div>
      </div>
  </div>
</template>

<script>
import { filterTask } from './filters/index.js';
import TodoForm from './components/TodoForm';
import Todos from './components/Todos';

export default {
    components: {
        TodoForm, Todos
    },
    data: function () {
        return {
            tasks: [],
            filter: 'all',
            newTask: ''
        }
    },
    events: {
        'task-create': function (title) {
            this.newTask = {
                id: 1,
                title: title,
                createdAt: Date.now(),
                finished: false,
            };
            this.$http.post('http://localhost:8081/api/todo',
              { todo: this.newTask  })
              .then(() => {
                this.tasks.push(this.newTask);
                this.newTask = '';
              }
            );
              this.fetchTasks();
        },
        'toggle-complete': function(data) {
          const finishedTask = data;
          console.log(finishedTask.finished);
          finishedTask.finished = ! data.finished;
          console.log(finishedTask.finished);
          this.$http.post('http://localhost:8081/api/todo/', { todo: finishedTask });
          this.tasks = this.tasks.map((task) => {
              if (task.id === data.id) {
                console.log('cycle',task.finished);
                task.finished = task.finished;
                console.log('cycle',task.finished);
                return task;
              }
            return task;
          });

        },
        'task-delete': function (data) {
          this.$http.delete('http://localhost:8081/api/todo/' + data.id)
              .then(() => {
                this.tasks = this.tasks.filter((task) => {
                  return task.id !== data.id;
                });
              }
            );
        }
    },
    methods: {
        filterTask(e) {
            const text = e.target.innerText;
            this.filter = text.toLowerCase();
        },
        fetchTasks() {
          this.$http.get('http://localhost:8081/api/todo')
            .then(function(response) {
              this.tasks = response.data;
          });
        },
    },
    created() {
      this.fetchTasks();
    },
}
</script>
