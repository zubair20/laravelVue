<template>
    <div class="container">
        <div class="row" v-if="$gate.isAdminOrAuthor()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add new <i class="fas fa-user-plus"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registerd at</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users.data" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at | myDate}}</td>
                      <td>
                          <a href="#" @click="editModal(user)" ><i class="fa fa-edit blue"></i></a>
                          <a href="#" @click="deleteUser(user.id)"><i class="fa fa-trash red"></i></a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->

              <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div class="row justify-content-center" v-if="!$gate.isAdminOrAuthor()">
            <div class="col-md-8">
                <not-found></not-found>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 v-show="!editMode" class="modal-title" id="addNewLabel">Add new</h5>
                    <h5 v-show="editMode" class="modal-title" id="addNewLabel">Update User's Info</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editMode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name" id="name" placeholder="Name"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email" id="email" placeholder="Email Address"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        <div class="form-group">
                            <textarea v-model="form.bio" id="bio" name="bio" placeholder="Short bio for User Optional"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>
                        <div class="form-group">
                            <select name="type" id="type" v-model="form.type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standart User</option>
                                <option value="author">Author</option>
                            </select>
                        </div>

                        <div class="form-group">
                            <input type="password" id="password" v-model="form.password" name="password" placeholder="Enter Password"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                    </div>
                
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="editMode" type="submit" class="btn btn-primary">Update</button>
                        <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                    </div>
                </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {

        data(){
            return {
                 editMode: false,
                 users : {},
                 form: new form({
                     'id': '',
                     'name': '',
                     'email': '',
                     'password': '',
                     'type': '',
                     'bio': '',
                     'photo': ''
                 })
            }
        },

        methods: {
            // Our method to GET results from a Laravel endpoint
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            editModal(user){
                this.editMode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editMode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            deleteUser(id){

                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {

                        if (result.value) {
                            this.form.delete('api/user/'+id)
                            .then(()=>{
                                Swal.fire(
                                    'Deleted!',
                                    'Your file has been deleted.',
                                    'success'
                                )

                                Fire.$emit('AfterCreate');
                            })
                            .catch(()=>{
                                Swal.fire({
                                    title: 'Error?',
                                    text: "You Cannot Deleted this!",
                                    type: 'error'
                                })
                            });
                        }
                })
            },
            loadUsers(){
                if (this.$gate.isAdminOrAuthor()) {
                    axios.get('api/user').then(({data}) => (this.users = data));
                }
                
            },
            updateUser(){
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    $('#addNew').modal('hide');
                    Swal.fire(
                        'Updated!',
                        'User has been Updated.',
                        'success'
                    )
                    this.$Progress.finish();
                    Fire.$emit('AfterCreate');

                })
                .catch(()=>{
                    this.$Progress.fail();
                });
            },
            createUser(){
                this.$Progress.start()
                this.form.post('api/user')
                .then(()=> {
                    Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');

                    Toast.fire({
                        type: 'success',
                        title: 'User created in successfully'
                    })
                    this.$Progress.finish()
                })
                .catch(()=>{

                });

                
            }
        },
        created() {
            Fire.$on('searching', () =>{
                let query = this.$parent.search;
                console.log(query);
                
                axios.get('api/findUser?q=' + query)
                .then((data)=> {
                    this.users = data.data
                    
                })
                .catch(()=>{
                    
                });
            })
            
            this.loadUsers();
            Fire.$on('AfterCreate', () =>{
                this.loadUsers();
            });
        }
    }
</script>
