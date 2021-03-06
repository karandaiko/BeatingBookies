<template>
  <div id="secure">
    <!-- Navbar Component -->
    <Navbar></Navbar>
    <section>
      <b-modal
        :active.sync="isAddModalActive"
        has-modal-card
        trap-focus
        aria-role="dialog"
        :can-cancel="['escape','outside']"
        aria-modal
        :height="60"
      >
        <AddBets></AddBets>
      </b-modal>
    </section>
    <section>
      <b-modal
        :active.sync="isEditModalActive"
        has-modal-card
        trap-focus
        aria-role="dialog"
        :can-cancel="['escape','outside']"
        aria-modal
        :height="60"
      >
        <EditBets v-bind:betData="betToEdit" v-bind:betType="betTypeToEdit"
        v-if="isEditModalActive">
        </EditBets>
      </b-modal>
    </section>
    <div class="columns">
      <div class="column is-2">
        <SideBar
        @view-bets="ViewBetsTable=true; ViewStats=false; ViewSettleBets=false"
        @view-stats="ViewStats=true; ViewBetsTable=false; ViewSettleBets=false"
        @view-dashboard="ViewBetsTable=true; ViewStats=true; ViewSettleBets=true"
        @add-bet="isAddModalActive=true">
        </SideBar>
      </div>
      <div class="container main-content column">
        <div class="tile is-ancestor" v-if="ViewStats">
          <div class="tile is-parent">
            <article class="tile is-child box notification is-info">
              <p class="title">Average conversion</p>
              <p class="subtitle">{{ average_conversion }}</p>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box notification is-success">
              <p class="title">Total profits</p>
              <p class="subtitle">{{ total_profits }}</p>
            </article>
          </div>
          <div class="tile is-parent">
            <article class="tile is-child box notification is-primary">
              <p class="title">Biggest win this week</p>
              <p class="subtitle">{{ biggest_win }}</p>
            </article>
          </div>
        </div>
        <div class="tile is-ancestor">
          <div class="tile is-8 is-vertical" v-if="ViewBetsTable">
            <div class="tile is-parent">
              <article class="tile is-child box is-info">
                <div class="level">
                  <p class="title level-right">Your bets ...</p>
                  <b-button
                    class="level-left button is-primary"
                    icon-left="plus"
                    @click="isAddModalActive = true"
                  >Add Bet</b-button>
                </div>
                <BetsTable v-bind:perPage="perPage" :isPaginated="true" v-bind:betData="betData"
                @update-bet-data="getBetData()" @edit-bet-data="editBetData">
                </BetsTable>
              </article>
            </div>
          </div>
          <div class="tile is-4 is-vertical" v-if="ViewSettleBets">
            <div class="tile is-parent">
              <article class="tile is-child box notification is-info">
                <p class="title">ToDo:Settle these bets !</p>
                <p class="subtitle">{{ average_conversion }}</p>
              </article>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>

<script>
import Navbar from '../components/Navbar.vue';
import BetsTable from '../components/BetsTable.vue';
import SideBar from '../components/SideBar.vue';
import Footer from '../components/Footer.vue';
import AddBets from '../components/AddBets.vue';
import EditBets from '../components/EditBets.vue';

export default {
  name: 'Secure',
  data() {
    return {
      isAddModalActive: false,
      isEditModalActive: false,
      isViewBetsTable: false,
      isViewStats: false,
      ViewStats: true,
      ViewBetsTable: true,
      ViewSettleBets: true,
      perPage: 5,
      betToEdit: {},
      server_msg: '',
      betTypeToEdit: '',
      betData: [],
    };
  },
  components: {
    Navbar,
    BetsTable,
    EditBets,
    AddBets,
    SideBar,
    Footer,
  },
  created() {
    this.getBetData();
  },
  methods: {
    getBetData() {
      this.$http
        .get(`${process.env.VUE_APP_SERVER_URL}bets/getAll`, {
          params: {
            user_id: JSON.parse(localStorage.getItem('user')).id,
          },
        })
        .then((response) => {
          this.betData = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    editBetData(bet) {
      Object.assign(this.betToEdit, bet);
      this.betTypeToEdit = bet.type;
      if (bet.type === this.betToEdit.type) this.isEditModalActive = true;
    },
  },
  computed: {
    total_profits() {
      const totalProfit = this.betData.reduce((accumulator, currentValue) => {
        if (!Number.isNaN(Number(currentValue.profits))) {
          return accumulator + currentValue.profits;
        }
        return accumulator;
      }, 0);
      return `AUD ${totalProfit}`;
    },
    biggest_win() {
      const biggestWin = this.betData.reduce((p, v) => {
        if (!Number.isNaN(Number(v.profits))) {
          return p > v.profits ? p : v.profits;
        }
        return p;
      }, 0);
      return `AUD ${biggestWin}`;
    },
    average_conversion() {
      const totalSuccess = this.betData.reduce((p, v) => {
        if (!Number.isNaN(Number(v.profits))) {
          return v.profits > 0 ? 1 + p : p;
        }
        return p;
      }, 0);
      const averageConversion = (totalSuccess / this.betData.length) * 100;
      return `${averageConversion} %`;
    },
  },
};
</script>

<style scoped>
.main-content {
  padding: 25px 20px 5px 0px;
}

.subtitle {
  font-size: "90px";
}
</style>
