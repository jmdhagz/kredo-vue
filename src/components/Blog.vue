<template>
	<div class="container">
		<!-- Login Form -->
		<div class="row mt-4" v-if="auth">
			<b-card class="col-md-6 offset-3">
				<b-card-text>
					<form action="#" @submit.prevent="userLogin">
						<h3>Sign in</h3>
						<div class="form-group mb-2">
							<input type="email" name="email" class="form-control" v-model="formData.email" placeholder="Email Address">
						</div>
						<div class="form-group mb-2">
							<input type="password" name="password" class="form-control" v-model="formData.password" placeholder="Password">
						</div>
						<div class="form-group mb-2">
							<button type="submit" class="btn btn-primary btn-block">Sign In</button>
						</div>
					</form>
				</b-card-text>
			</b-card>
		</div>
		<!-- Blog List -->
		<div class="row mt-4" v-if="!auth">
			<div class="col-6 offset-3">
				<h3>Blog List</h3>
				<b-button variant="primary" v-b-modal.create-modal>Create new</b-button>

				<!-- Create Modal -->
				<b-modal id="create-modal" hide-footer title="Create Blog">
					<form action="#" @submit.prevent="createBlog">
						<div class="form-row mb-2">
							<label>Title</label>
							<input type="text" name="title" class="form-control" v-model="newBlog.title" placeholder="Title">
						</div>
						<div class="form-row mb-2">
							<label>Description</label>
							<input type="text" name="description" class="form-control" v-model="newBlog.description" placeholder="Description">
						</div>
						<div class="form-row mb-2">
							<button type="submit" class="btn btn-primary btn-block">Save</button>
						</div>
					</form>
				</b-modal>

				<ul class="list-group mt-2" v-for="(blog, index) in blogList" :key="index">
					<li class="list-group-item">
						<h3>{{ blog.title }}</h3>
						<p class="lead" style="margin-bottom: 0px;">{{ blog.description }}</p>
						<b-button v-b-modal="'modal-'+blog.id">Edit</b-button>&nbsp;
						<button class="btn btn-danger btn-block" @click="deleteBlog(blog.id)">Delete</button>

						<!-- Edit Modal -->
						<b-modal :id="'modal-'+blog.id" hide-footer title="Edit Blog">
							<form action="#" @submit.prevent="editBlog(blog)">
								<div class="form-row mb-2">
									<label>Title</label>
									<input type="text" name="title" class="form-control" v-model="blog.title" placeholder="Title">
								</div>
								<div class="form-row mb-2">
									<label>Description</label>
									<input type="text" name="description" class="form-control" v-model="blog.description" placeholder="Description">
								</div>
								<div class="form-row mb-2">
									<button type="submit" class="btn btn-primary btn-block">Save Changes</button>
								</div>
							</form>
						</b-modal>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				blogList: [],
				formData: {
					email: '',
					password: ''
				},
				newBlog: {
					'title': '',
					'description': ''
				},
				token: '',
				auth: true
			}
		},
		methods: {
			userLogin(){
				this.axios.get('http://localhost/laravel-sanctum/public/sanctum/csrf-cookie')
				.then(response=>{
					console.log(response)
					this.axios.post('http://localhost/laravel-sanctum/public/api/login', this.formData).then(response => {
						localStorage.setItem('user-token', response.data.token);
						this.getBlogs();
						this.auth = false;
					});
				});
			},
			getConfig() {
				let config = {
					headers: {
						'Authorization': 'Bearer '+ localStorage.getItem('user-token')
					}
				}
				return config;
			},
			getBlogs() {
				this.axios.get('http://localhost/laravel-sanctum/public/api/blogs', this.getConfig()).then(response => {
					console.log(response);
					this.blogList = response.data;
				});
			},
			createBlog() {
				this.axios.post('http://localhost/laravel-sanctum/public/api/blogs/create', this.newBlog, this.getConfig())
				.then(response => {
					console.log(response);
					this.getBlogs();
					this.$bvModal.hide('create-modal')
				});
			},
			editBlog(blog) {
				this.axios.post('http://localhost/laravel-sanctum/public/api/blogs', blog, this.getConfig())
				.then(response => {
					console.log(response);
					this.$bvModal.hide('modal-'+blog.id)
					this.getBlogs();
				});
			},
			deleteBlog(id) {
				this.axios.post('http://localhost/laravel-sanctum/public/api/blogs/'+id, {}, this.getConfig())
				.then(response => {
					console.log(response);
					this.getBlogs();
				});
			}
		}
	}
</script>