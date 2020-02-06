Task.vue:

<template>
	<div class="container">
		<div class="row">
			<div class="col-md-12">
				<div class="panel panel-default">
					<div class="panel-heading">
						<h3><i class="fa fa-tachometer"></i>Task Dashboard</h3> <br>
						<button @click="initAddTask()" class="btn btn-success" style="padding:5px;margin-bottom:10px">
							Add New Task
						</button>
					</div>
					<div class="panel-body">
						<table style="display:table" class="table table-bordered table-striped table-responsive" v-if="tasks.length > 0">
							<tbody>
								<tr>
									<th style="width:40px;text-align:center;">No.</th>
									<th class="">Name</th>
									<th class="">Description</th>
									<th style="width:85px;text-align:center;">Action</th>
								</tr>
								<tr v-for="(task, index) in tasks">
									<td style="text-align:center;">{{ index + 1}}</td>
									<td>{{ task.name }}</td>
									<td>{{ task.description }}</td>
									<td style="text-align:center;">
										<button @click="initUpdate(index)" title="Edit" class="btn btn-success btn-xs" style="padding:2px 6px"><i class="fa fa-pencil"></i></button>
										<button @click="deleteTask(index)" title="Delete" class="btn btn-danger btn-xs"  style="padding:2px 6px"><i class="fa fa-trash"></i></button>
									</td>
								</tr>
							</tbody>
						</table>
					</div>
				</div>
			</div>
		</div>
		
		<div class="modal fade" tabindex="-1" role="dialog" id="add_task_model">
			<div class="modal-dialog" role="document">
				<div class="modal-content">
					<div class="modal-header">
						<h4 class="modal-title">Add New Task</h4>
						<button type="button" class="close" data-dismiss="modal" aria-label="Close">
							<span aria-hidden="true">&times;</span>
						</button>
					</div>
					<div class="modal-body">
						<div class="alert alert-danger" v-if="errors.length > 0">
							<ul>
								<li v-for="error in errors">{{ error }}</li>
							</ul>
						</div>
						<div class="form-group">
							<label for="names">Name:</label>
							<input type="text" name="name" id="name" placeholder="Task Name" class="form-control" v-model="task.name">
						</div>
						<div class="form-group">
							<label for="description">Description:</label>
							<textarea name="description" id="description" cols="30" rows="5" class="form-control" placeholder="Task Description" v-model="task.description"></textarea>
						</div>
					</div>
					<div class="modal-footer">
						<button class="btn btn-default" type="button" data-dismiss="modal">Close</button>
						<button type="button" @click="createTask" class="btn btn-primary">Submit</button>
					</div>
				</div><!-- /.modal-content -->
			</div><!-- /.modal-dialog -->
		</div><!-- /.modal -->
		
		<div class="modal fade" tabindex="-1" role="dialog" id="update_task_model">
			<div class="modal-dialog" role="document">
				<div class="modal-content">
					<div class="modal-header">
						<h4 class="modal-title">Update Task</h4>
						<button type="button" class="close" data-dismiss="modal" aria-label="Close">
							<span aria-hidden="true">&times;</span>
						</button>
					</div>
					<div class="modal-body">
						<div class="alert alert-danger" v-if="errors.length > 0">
							<ul>
								<li v-for="error in errors">{{ error }}</li>
							</ul>
						</div>
						<div class="form-group">
							<label for="name">Name:</label>
							<input type="text" placeholder="Task Name" class="form-control" v-model="update_task.name">
						</div>
						<div class="form-group">
							<label for="description">Description:</label>
							<textarea name="description" id="descriptiom" cols="30" rows="5" class="form-control" placeholder="Task Description" v-model="update_task.description"></textarea>
						</div>
					</div>
					<div class="modal-footer">
						<button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
						<button type="button" @click="updateTask" class="btn btn-primary">Submit</button>
					</div>
				</div><!-- /.modal-content -->
			</div><!-- /.modal-dialog -->
		</div><!-- /.modal -->

	</div>
</template>

<script>
	export default {

		data() {
			return {
				task: {
					name: '',
					description: ''
				},
				errors:[],
				tasks:[],
				update_task:{}
			}
		},

		mounted() {
			this.readTasks();
		},

		methods: {

			deleteTask(index) {
				let conf = confirm('Do you really want to delete this task?');
				if (conf === true) {
					axios.delete('/task/' + this.tasks[index].id)
					.then(response => {
						this.tasks.splice(index, 1);
					})
					.catch(error => {

					});
				}
			},

			initAddTask() {
				$('#add_task_model').modal('show');
			},

			createTask() {
				axios.post('/task/', {
					name: this.task.name,
					description: this.task.description,
				})
				.then(response => {
					this.reset();
					this.tasks.push(response.data.task);
					$('#add_task_model').modal('hide');
				})
				.catch(error => {
					this.errors = [];
					if(error.response.data.errors && error.response.data.errors.name) {
						this.errors.push(error.response.data.errors.name[0]);
						if(error.response.data.errors && error.response.data.errors.description)
						this.errors.push(error.response.data.errors.description[0]);
					}
				});
			},

			reset() {
				this.task.name = '';
				this.task.description = '';
			},

			readTasks() {
				axios.get('http://localhost:8000/task')
				.then(response => {
					this.tasks = response.data.tasks;
				});
			},

			initUpdate(index) {
				this.errors = [];
				$('#update_task_model').modal('show');
				this.update_task = this.tasks[index];
			},

			updateTask() {
				axios.patch('/task/' + this.update_task.id, {
					name: this.update_task.name,
					description: this.update_task.description,
				})
				.then(response => {
					$('#update_task_model').modal('hide');
				})
				.catch(error => {
					this.errors = [];
					if(error.response.data.errors.name) {
						this.errors.push(error.response.data.errors.name[0]);
					}
					if(error.response.data.errors.description) {
						this.errors.push(error.response.data.errors.description[0]);
					}
				});
			}
		}
	}
</script>