<template>
  <category-detail
    :operationType="operation"
    :category="category"
    :errors="errors"
    @save="save"
    @cancel="cancel"
  ></category-detail>
</template>

<script>
import CategoryDetail from "./CategoryDetail.vue"

export default {
  name: "Category",
  components: {
    CategoryDetail,
  },
  props: {
    id: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      category: this.newCategory(),
      errors: null,
    }
  },
  computed: {
    operation() {
      return !this.id || this.id < 0 ? "insert" : "update"
    },
  },
  watch: {
    id: {
      immediate: true,
      handler(newValue) {
        this.loadCategory(newValue)
      },
    },
  },
  methods: {
    newCategory() {
      return {
        id: null,
        vcard: this.$store.state.user ? this.$store.state.user.username : "",
        type: "C",
        name: "",
      }
    },
    loadCategory(id) {
      if (!id || id < 0) {
        this.category = this.newCategory()
      } else {
        this.$axios
          .get("categories/" + id)
          .then((response) => {
            this.category = response.data.data
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    save() {
      if (this.operation == "insert") {
        this.$axios
          .post("categories", this.category)
          .then((response) => {
            this.$toast.success(
              "Category #" +
                response.data.data.id +
                " was created successfully."
            )
            this.$router.back()
          })
          .catch((error) => {
            if (error.response.status == 422) {
              this.errors = error.response.data.errors
              this.$toast.error(
                "Category was not created due to validation errors!"
              )
            } else {
              this.$toast.error(
                "Category was not created due to unknown server error!"
              )
            }
          })
      } else {
        this.$axios
          .put("categories/" + this.id, this.category)
          .then((response) => {
            this.$toast.success(
              "Category #" +
                response.data.data.id +
                " was updated successfully."
            )
            this.$router.back()
          })
          .catch((error) => {
            if (error.response.status == 422) {
              this.$toast.error(
                "Category #" +
                  this.id +
                  " was not updated due to validation errors!"
              )
            } else {
              this.$toast.error(
                "Category #" +
                  this.id +
                  " was not updated due to unknown server error!"
              )
            }
          })
      }
    },
    cancel() {
      this.$router.back()
    },
  },
}
</script>
