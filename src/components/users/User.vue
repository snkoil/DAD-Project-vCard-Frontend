<template>
  <user-detail
    :user="user"
    @save="save"
    @cancel="cancel"
    :errors="errors"
  ></user-detail>
</template>

<script>
import UserDetail from "./UserDetail.vue"

export default {
  name: 'User',
  components: {
    UserDetail
  },
  props: {
    id: {
      type: Number,
      default: null
    },
  },
  data () {
    return {
      user: this.newUser(),
      errors: null
    }
  },
  watch: {
    // beforeRouteUpdate was not fired correctly
    // Used this watcher instead to update the ID
    id: {
      immediate: true,
      handler (newValue) {
        this.loadUser(newValue)
      }
    },
  },
  methods: {
    newUser () {
      return {
        id: null,
        name: '',
        email: '',
        photo_url: null
      }
    },
    loadUser (id) {
      if (!id || (id < 0)) {
        this.user = this.newUser()
      } else {
        this.$axios.get('administrators/' + id)
          .then((response) => {
            this.user = response.data.data
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    save () {
      this.$axios.put('administrators/' + this.id, this.user)
        .then((response) => {
          this.$toast.success('User #' + response.data.data.id + ' was updated successfully.')
          this.$router.back()
        })
        .catch((error) => {
          if (error.response.status == 422) {
            this.errors = error.response.data.errors
            console.log(error)
            this.$toast.error('User #' + this.id + ' was not updated due to validation errors!')
          } else {
            this.$toast.error('User #' + this.id + ' was not updated due to unknown server error!')
          }
        })
    },
    cancel () {
      // Replace this code to navigate back
      // this.loadUser(this.id)
      this.$router.back()
    }
  },
  mounted () {
  }
}
</script>
