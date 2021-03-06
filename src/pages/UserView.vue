<template>
  <!-- eslint-disable -->
  <layout :source="title">
    <template slot="body">
      <div class="container p-0" v-show="loaded">
        <div class="row user-display m-4 p-4">
          <div class="col-12 p-0">
            <form @submit.prevent="update">
              <div class="form-group row">
                <label for="id" class="col-sm-2 col-form-label">Id</label>
                <div class="col-sm-10">
                  <input type="text" disabled class="form-control" id="id" v-model="user.id" placeholder="Id">
                </div>
              </div>
              <div class="form-group row">
                <label for="username" class="col-sm-2 col-form-label">Username</label>
                <div class="col-sm-10">
                  <input type="text" disabled class="form-control" id="username" v-model="user.username" placeholder="Username">
                </div>
              </div>
              <div class="form-group row">
                <label for="name" class="col-sm-2 col-form-label">Name</label>
                <div class="col-sm-10">
                  <input type="text" :disabled="!canChange()" required class="form-control" id="name" v-model="user.name" placeholder="Name">
                </div>
              </div>
              <div class="form-group row">
                <label for="surname" class="col-sm-2 col-form-label">Surname</label>
                <div class="col-sm-10">
                  <input type="text" :disabled="!canChange()" required class="form-control" id="surname" v-model="user.surname" placeholder="Surname">
                </div>
              </div>
              <div class="form-group row">
                <label for="email" class="col-sm-2 col-form-label">Email</label>
                <div class="col-sm-10">
                  <input type="email" :disabled="!canChange()" required class="form-control" id="email" v-model="user.email" placeholder="Email">
                </div>
              </div>
              <div class="row">
                <div class="col-12 pl-0 pr-0">
                  <div class="alert alert-danger" role="alert" v-if="error">
                    Unable to change user. Please verify the data and retry.
                  </div>
                </div>
              </div>
              <div class="row">
                <div class="col-12 ">
                  <div class="d-flex flex-row-reverse">
                    <router-link v-if="!canChange()" :to="{name: 'users'}" class="btn btn-secondary">Back</router-link>
                    <button v-if="canChange()" class="btn btn-primary" type='submit'>Save</button>
                    <router-link v-if="canChange()" :to="{name: 'users'}" class="btn btn-secondary mr-3">Cancel</router-link>
                    <router-link v-if="canChange() && isSelf()" :to="{name: 'user-change-password', params: {id: user.id}}" class="btn btn-dark mr-3">Change password</router-link>
                    <router-link v-if="canChange() && !isSelf() && isAdmin" :to="{name: 'user-reset-password', params: {id: user.id}}" class="btn btn-danger mr-3">Reset password</router-link>
                  </div>
                </div>
              </div>
            </form>
          </div>
        </div>
      </div>
      <loader :visible="!loaded" :label="'Loading user data...'"/>
      <loader :visible="saving" :label="'Saving users...'"/>
    </template>
  </layout>
</template>

<script>
import Layout from '../components/Layout'
import Loader from '../components/Loader'
import router from '../router/index'
import { updateUser, getUser } from '../service/apiService'
import { mapGetters } from 'vuex'

export default {
  components: {
    Layout,
    Loader
  },
  name: 'User',
  data() {
    return {
      title: 'User',
      user: {
        id: '',
        name: '',
        surname: '',
        email: ''
      },
      loaded: false,
      saving: false,
      error: false
    }
  },
  computed: {
    ...mapGetters('login', [
      'getUsername',
      'isAdmin'
    ])
  },
  mounted() {
    getUser(this.$route.params.id).then((user) => {
      Object.assign(this.user, user)
      this.$set(this, 'loaded', true)
    })
  },
  methods: {
    update() {
      this.saving = true
      this.error = false
      updateUser(this.user)
        .then(() => {
          router.push({ name: 'users' })
          this.saving = false
        })
        .catch(() => {
          this.saving = false
          this.error = true
        })
    },
    canChange() {
      return this.isAdmin || this.isSelf()
    },
    isSelf() {
      return this.getUsername === this.user.username
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  div.form-group {
    margin-bottom: 1rem;
  }

  input {
    font-size: 1em;
  }

  .btn {
    font-size: 1em;
  }
</style>
