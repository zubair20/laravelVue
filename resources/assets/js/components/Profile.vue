<template>
    <div class="container">
        <div class="row mt-3">
            <div class="col-md-12">
                <div class="card card-widget widget-user">
                    <!-- Add the bg color to the header using any of the bg-* classes -->
                    <div class="widget-user-header text-white" style="background: url('/img/photo1.jpg') center center;">
                        <h3 class="widget-user-username text-right">Elizabeth Pierce</h3>
                        <h5 class="widget-user-desc text-right">Web Designer</h5>
                    </div>
                    <div class="widget-user-image">
                        <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar">
                    </div>
                    <div class="card-footer">
                        <div class="row">
                        <div class="col-sm-4 border-right">
                            <div class="description-block">
                            <h5 class="description-header">3,200</h5>
                            <span class="description-text">SALES</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        <div class="col-sm-4 border-right">
                            <div class="description-block">
                            <h5 class="description-header">13,000</h5>
                            <span class="description-text">FOLLOWERS</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        <div class="col-sm-4">
                            <div class="description-block">
                            <h5 class="description-header">35</h5>
                            <span class="description-text">PRODUCTS</span>
                            </div>
                            <!-- /.description-block -->
                        </div>
                        <!-- /.col -->
                        </div>
                        <!-- /.row -->
                    </div>
                </div>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header p-2">
                        <ul class="nav nav-pills">
                        <li class="nav-item"><a class="nav-link" href="#">Activity</a></li>
                        <li class="nav-item"><a class="nav-link active" href="#settings" data-toggle="tab">Settings</a></li>
                        </ul>
                    </div><!-- /.card-header -->
                    <div class="card-body">
                        <div class="tab-pane active" id="settings">
                            <form class="form-horizontal">
                                <div class="form-group row">
                                    <label for="name" class="col-sm-2 col-form-label">Name</label>
                                    <div class="col-sm-10">
                                        <input v-model="form.name" name="name" type="text" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }" id="name" placeholder="Name">
                                        <has-error :form="form" field="name"></has-error>
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <label for="email" class="col-sm-2 col-form-label">Email</label>
                                    <div class="col-sm-10">
                                        <input v-model="form.email" name="email" type="email" class="form-control" :class="{ 'is-invalid': form.errors.has('email') }" id="email" placeholder="Email">
                                        <has-error :form="form" field="email"></has-error>
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <label for="inputExperience" class="col-sm-2 col-form-label">Experience</label>
                                    <div class="col-sm-10">
                                    <textarea class="form-control" id="inputExperience" placeholder="Experience"></textarea>
                                    </div>
                                </div>
                                
                                <div class="form-group row">
                                    <label for="photo" class="col-sm-2 col-form-label">Profile Photo</label>
                                    <div class="col-sm-10">
                                        <input type="file" @change="updateProfile" id="photo" name="photo">
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <label  for="password" class="col-sm-2 col-form-label">Password</label>
                                    <div class="col-sm-10">
                                        <input v-model="form.password" type="password" class="form-control" :class="{ 'is-invalid': form.errors.has('password') }" id="password" placeholder="Password (Leave empty if not changing)">
                                        <has-error :form="form" field="password"></has-error>
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <div class="offset-sm-2 col-sm-10">
                                    <button type="submit" @click.prevent="updateInfo" class="btn btn-success">Submit</button>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </div><!-- /.card-body -->
            </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){

            return {
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
        mounted() {
            console.log('Component mounted.')
        },

        methods: {
            getProfilePhoto(){
                let photo = (this.form.photo.length > 100) ? this.form.photo : "img/profile/"+this.form.photo;
                return photo;
            },
            updateInfo(){
                this.$Progress.start();
                if (this.form.password == '') {
                    this.form.password = undefined;
                }
                this.form.put('api/profile')
                .then(()=>{

                    Fire.$emit('AfterCreate');
                    this.$Progress.finish();
                })
                .catch(()=>{
                    this.$Progress.fail();
                });
            },
            updateProfile(e){
                let file = e.target.files[0];
                console.log(file);
                let reader = new FileReader();
                
                if (file['size'] < 2111775) {
                    reader.onloadend = (file) => {

                        console.log('Result', reader.result);
                        
                        this.form.photo = reader.result;
                    }
                    reader.readAsDataURL(file);
                }else{

                     Swal.fire({
                         type: 'error',
                         title: 'Ooops....',
                         text: 'You are Uploading large file'
                     })
                }
                
                
            }
        },
        created(){

            axios.get('api/profile')
            .then(({data}) => (this.form.fill(data)));
        }
    }
</script>
