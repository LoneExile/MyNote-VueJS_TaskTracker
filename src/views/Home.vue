<template>
    <!-- v-if -->
    <div v-show="showAddTask">
        <!-- you can move v-show="showAddTask" to <AddTask />  -->
      <AddTask @add-task="addTask" />
    </div>
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
    
</template>

<script>
import Tasks from '@/components/Tasks.vue';
import AddTask from '@/components/AddTask.vue';

export default {
    name: 'Home',
    props: {
        showAddTask: Boolean,
    },
    components: {
        Tasks,
        AddTask,
    },
    data() {
        return {
            tasks: [],
        }
    },
    methods: {
    // task is newTask from [this.$emit('add-task', newTask)] in AddTask.vue
        async addTask(task) {
        const res = await fetch('api/tasks', {
            method: 'POST',
            headers: {
            'Content-Type': 'application/json',
            },
            body: JSON.stringify(task)
        })
        const data = await res.json()
        this.tasks = [...this.tasks,data]
        },
        async deleteTask(id) {
        //  console.log('task',id); 
        if (confirm('Are you sure you want to delete')) {
            const res = await fetch(`api/tasks/${id}`, {
            method: 'DELETE'
            })
            res.status === 200 ? (this.tasks = this.tasks.filter((task) => task.id !== id)): alert('Error deleting task');
            // filter((task)) want anything except this id
        }   
        },
        async toggleReminder (id) {
        const taskToToggle = await this.fetchTask(id)
        const updTask = {...taskToToggle, reminder:!taskToToggle.reminder}
        const res = await fetch(`api/tasks/${id}`, {
            method: 'PUT',
            headers: {
            'Content-type': 'application/json'
            },
            body: JSON.stringify(updTask)
        })
        const data = await res.json()
        //console.log('toggle',id);
        // each(task) if(task.id = id) {return change reminder to opposite val} else {if not match return task}
        
        //console.log(this.tasks[0].id);
        //console.log(...this.tasks);
        this.tasks = this.tasks.map((task) => task.id === id? {...task, reminder: data.reminder} : task)
        },
        async fetchTasks() {
        const res = await fetch('api/tasks');
        const data = await res.json()
        return data
        },
        async fetchTask(id) {
        const res = await fetch(`api/tasks/${id}`);
        const data = await res.json()
        return data
        }
    },
    
    async created () {
        this.tasks = await this.fetchTasks()
    },
}
</script>

<style scoped>

</style>