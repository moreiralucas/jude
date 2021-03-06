<template>
  <div :class="{ 'sidemenu-active': isSideMenuActive }">
    <div class="sidemenu">
      <ul>
        <li><router-link class="sidemenu-item" to="dashboard">Dashboard</router-link></li>
        <li><router-link class="sidemenu-item" to="standings">Standings</router-link></li>
        <li><router-link class="sidemenu-item" v-if="isAdmin()" to="settings">Settings</router-link></li>
        <hr class="rule">
        <li><span class="sidemenu-item ju-secondary-text">{{ countdownString }}</span></li>
        <li><a class="sidemenu-item" @click="doLogout()">Logout</a></li>
      </ul>
    </div>

    <nav class="navbar">
      <div class="navbar-brand">
        <a class="navbar-item">
          <img src="/static-jude/images/crown-48.png" width="48" height="38">
        </a>
        <div class="navbar-burger burger" @click="isSideMenuActive = !isSideMenuActive">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
      <div class="navbar-menu">
        <div class="navbar-start">
          <router-link class="navbar-item" to="dashboard">
            <b-icon icon="dashboard" size="is-small"></b-icon>
            <span>Dashboard</span>
          </router-link>
          <router-link class="navbar-item" to="standings">
            <b-icon icon="list-ol" size="is-small"></b-icon>
            <span>Standings</span>
          </router-link>
          <router-link v-if="isAdmin()" class="navbar-item" to="settings">
            <b-icon icon="cogs" size="is-small"></b-icon>
            <span>Settings</span>
          </router-link>
        </div>

        <div class="navbar-end">
          <div class="navbar-item" v-if="!isAdmin()">
            <div class="field">
              <p class="control">
                <a class="button is-primary" @click="submitModal.active = true">
                  <span class="icon"><b-icon icon="send" size="is-small"></b-icon></span>
                  <span>Submit</span>
                </a>
              </p>
            </div>
          </div>
          <div class="navbar-item ju-secondary-text">
            {{ countdownString }}
          </div>
          <div class="navbar-item has-dropdown is-hoverable">
            <a class="navbar-link">
              <span class="icon"><b-icon icon="user" size="is-small"></b-icon></span>
              <span>Control Panel</span>
            </a>
            <div class="navbar-dropdown">
              <b-switch class="navbar-item"
                v-model="config.autoFetchStandings"
                @input="setAutoFetch">Auto-refresh</b-switch>
              <a v-if="isAdmin()"
                class="navbar-item" @click="extractGhosts()">Extract Ghosts</a>
              <a class="navbar-item" @click="doLogout()">Logout</a>
            </div>
          </div>
        </div>
      </div>
    </nav>
    <div class="indeterminate" :class="{ active: isFetching }"></div>
    <section class="section main-section">
      <router-view></router-view>
    </section>

    <b-modal
      :component="SubmitComponent"
      :active.sync="submitModal.active">
    </b-modal>
  </div>
</template>

<script type="text/babel">import * as Api from "./api.js";
    import * as Helper from "./helpers.js";
    import SubmitComponent from "./submit.vue";
    import GhostExtractor from "./admin/ghosts.js";
    import Vue from "vue";
    import moment from "moment";
    import "moment/locale/en-gb";
    import { mapGetters, mapState } from "vuex";
    import { types } from "./store/";

    moment.locale("en-gb");

    export default {
      mounted() {
        this.fetch();
        this.countdownTimer = window.setInterval(() => {
          this.countdownString = this.getRemainingTime();
        }, 1000);
        this.fetchTimer = window.setInterval(async () => this.autoFetch(), 5000);
      },
      beforeDestroy() {
        if (this.fetchTimer)
          window.clearInterval(this.fetchTimer);
        if (this.countdownTimer)
          window.clearInterval(this.countdownTimer);
      },
      data() {
        return {
          isSideMenuActive: false, 
          countdownString: "-",
          SubmitComponent,
          fetchTimer: null,
          countdownTimer: null,
          submitModal: {
            active: false
          }
        };
      },
      computed: {
        ...Helper.mapModuleState("main", [
          "user",
          "userObject",
          "rawContest",
          "rawSubmissions",
          "rawTeams",
          "config"
        ]),
        ...mapGetters([
          "problems",
          "my",
          "groupedSubs",
          "teams",
          "languages",
          "submissions",
          "getRawProblem",
          "isFetching"
        ])
      },
      methods: {
        extractGhosts() {
          const win = window.open("", "_blank");
          const code = new GhostExtractor(this.rawContest, this.teams,
            this.problems, this.submissions).extract({tag: true});
          win.document.title = `Ghosts for Codeforces`;
          win.document.body.innerHTML = `<pre>${code}</pre>`;
        },
        getSelf() {
          return this.userObject;
        },
        isAdmin() {
          return Helper.isAdmin(this.getSelf());
        },
        getRemainingTime() {
          return Helper.getRemainingTime(this.rawContest);
        },
        getContestName() {
          const contest = this.rawContest;
          if (!contest)
            return "-";
          return contest.name;
        },
        getStartTime() {
          const contest = this.rawContest;
          if (!contest)
            return "-";

          return moment(contest.start_time).format("LLL (Z)");
        },
        getEndTime() {
          const contest = this.rawContest;
          if (!contest)
            return "-";
          return moment(contest.end_time).format("LLL (Z)");
        },
        hasStarted() {
          return Helper.hasContestStarted(this.rawContest);
        },
        async fetch() {
          try {
            await this.$store.dispatch(types.FETCH_CONTEST_DATA);
          } catch (err) {
            this.$jude.dealWithResponse(err);
            this.$store.commit(types.SET_AUTO_FETCH_STANDINGS, false);
          }
        },
        async doLogout() {
          this.$jude.logout();
        },
        async autoFetch() {
          if (this.config.autoFetchStandings)
            await this.fetch();
        },
        setAutoFetch(val, $event) {
          this.$store.commit(types.SET_AUTO_FETCH_STANDINGS, val);
        }
      },
      components: { 
        //JuSubmit: SubmitComponent,
        //JuCodeModal: CodeModalComponent
      }
    };
</script>

<style lang="sass">
</style>
