<template>
  <div class="d-flex justify-content-between">
    <div class="mx-2">
      <h3 class="mt-4">VCards</h3>
    </div>
    <div class="mx-2 total-filtro">
      <h5 class="mt-4">Total: {{ totalVCards }}</h5>
    </div>
  </div>
  <hr />
  <div class="mb-3 d-flex justify-content-between flex-wrap"></div>
  <VCardsTable
    :vcards="vCards"
    @toggleBlock="toggleBlockVCard"
    @edit="editVCard"
    @createTransaction="createTransaction"
  ></VCardsTable>
  <template class="paginator">
    <pagination
      v-model="page"
      :records="paginationData ? paginationData.total : 0"
      :per-page="paginationData ? paginationData.per_page : 0"
      @paginate="loadVCards"
      :options="{ hideCount: true, theme: 'bootstrap3' }"
    ></pagination>
  </template>
</template>

<script>
import VCardsTable from "./VCardsTable.vue"

export default {
  name: "VCards",
  components: {
    VCardsTable,
  },
  data() {
    return {
      vCards: [],
      page: 1,
      paginationData: null,
    }
  },
  computed: {
    totalVCards() {
      return this.paginationData ? this.paginationData.total : 0
    },
  },
  methods: {
    loadVCards() {
      this.$axios
        .get("vcards?page=" + this.page)
        .then((response) => {
          this.vCards = response.data.data
          this.paginationData = response.data.meta
        })
        .catch((error) => {
          console.log(error)
          this.vCards = []
        })
    },
    addVCard() {
      this.$router.push({ name: "NewVCard" })
    },
    toggleBlockVCard(vcard) {
      this.$axios
        .patch("vcards/" + vcard.phone_number + "/blocked", {
          blocked: vcard.blocked ? 0 : 1,
        })
        .then((response) => {
          if (vcard.blocked == 0) {
            this.$socket.emit("userBlocked", vcard.phone_number)
          }
          let receivedVCard = response.data.data
          let phone_numberIndex = this.vCards.findIndex(
            (t) => t.phone_number == receivedVCard.phone_number
          )
          if (phone_numberIndex >= 0) {
            this.vCards[phone_numberIndex].blocked = receivedVCard.blocked
            this.$toast.success(
              "VCard #" +
                vcard.phone_number +
                " was " +
                (receivedVCard.blocked ? "locked" : "unlocked") +
                " successfully."
            )
          }
        })
        .catch((error) => {
          this.$toast.error(
            "VCard #" +
              vcard.phone_number +
              " was " +
              (vcard.blocked ? "unlocked" : "locked") +
              " unsuccessfully."
          )
          console.log(error)
        })
    },
    editVCard(vcard) {
      this.$router.push({
        name: "VCard",
        params: {
          id: vcard.phone_number,
        },
      })
    },
    createTransaction(vcard){
      this.$router.push({
        name: "Create Credit Transaction",
        params: {
          vcard: vcard.phone_number,
        },
      })
    },
  },
  mounted() {
    this.loadVCards()
  },
}
</script>

<style scoped>
.filter-div {
  min-wphone_numberth: 12rem;
}
.total-filtro {
  margin-top: 0.35rem;
}
.btn-addprj {
  margin-top: 1.85rem;
}
.paginator {
  display: flex;
  justify-content: center;
}
</style>
