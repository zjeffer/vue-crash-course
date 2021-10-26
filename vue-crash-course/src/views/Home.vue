<template>
	<AddTask v-show="showAddTask" @add-task="addTask" />
	<Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
	name: "Home",
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
		};
	},

	methods: {
		async addTask(task) {
			const res = await fetch("api/tasks", {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify(task),
			});

			const data = await res.json();

			this.tasks = [...this.tasks, data];
		},
		async deleteTask(id) {
			if (confirm("Are you sure?")) {
				const res = await fetch(`api/tasks/${id}`, {
					method: "DELETE",
				});

				res.status == 200
					? (this.tasks = this.tasks.filter((task) => task.id !== id))
					: alert("Something went wrong");
			}
		},
		async toggleReminder(id) {
			const taskToToggle = await this.fetchTask(id);
			const updTask = {
				...taskToToggle,
				reminder: !taskToToggle.reminder,
			};

			const res = await fetch(`api/tasks/${id}`, {
				method: "PUT",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify(updTask),
			});

			const data = await res.json();

			this.tasks = this.tasks.map((task) => (task.id === id ? { ...task, reminder: data.reminder } : task));
		},

		async fetchTasks() {
			const response = await fetch("api/tasks");
			return await response.json();
		},
		async fetchTask(id) {
			const response = await fetch(`api/tasks/${id}`);
			return await response.json();
		},
	},
	async created() {
		this.tasks = await this.fetchTasks();
	},
};
</script>
