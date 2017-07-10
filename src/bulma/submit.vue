<template>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Submit</p>
      </header>
      <section class="modal-card-body">
        <b-field label="Problem">
          <b-select placeholder="Select a problem" v-model="problem" required expanded>
            <option
              v-for="prob in problems"
              :value="prob.problem._id"
              :key="prob.problem._id">
              {{ prob.letter }}. {{ prob.problem.name }}
            </option>
          </b-select>
        </b-field>
        <b-field label="Language">
          <b-select placeholder="Select a language" v-model="language" required expanded>
            <option
              v-for="lang in languages"
              :value="lang[0]"
              :key="lang[0]">
              {{ lang[1] }}
            </option>
          </b-select>
        </b-field>
        <b-field label="Code">
          <brace
            :id="braceId"
            v-model="code"
            style="height: 175px;"
            :theme="'github'" :mode="getMode()"></brace>
        </b-field>
      </section>
      <footer class="modal-card-foot">
        <a class="button is-success" @click="submit()">Submit Code</a>
        <a class="button" @click="close()">Close</a>
      </footer>
    </div>
</template>

<script type="text/babel">
    // import 'babel-polyfill';
    import * as Helper from './helpers.js';
    import * as Api from './api.js';
    import { mapGetters } from "vuex";
    import { types } from "./store/";
    import BulmaUtils from "./bulmutils";
    import Brace from "./components/Brace.vue";

    function genRandom(len) {
      const data = "0123456789abcdef";
      let res = "";
      for (let i = 0; i < len; i++)
        res += data[(Math.random() * data.length)|0];
      return res;
    }

    export default {
        mounted () {
          this.braceId = genRandom(8);
        },
        data() {
            return {
                submitting: 0,
                braceId: "default"
            }
        },
        props: {
          problem: {
            default: null
          },
          language: {
            default: null,
            type: String
          },
          code: {
            default: "",
            type: String
          }
        },
        computed: {
            ...mapGetters([
                "problems",
                "languages"
            ])
        },
        methods:{
            getProblem(id){
                for (let prob of this.problems) {
                    if (prob.problem._id === id)
                        return prob;
                }

                return undefined;
            },
            close() {
              this.$emit("close");
            },
            submit(){
                if(this.submitting++ > 0) {
                    this.submitting--;
                    return;
                }

                const { problem, code, language } = this;

                Api.submit.save({
                    problem,
                    code,
                    language
                }).then(async (result) => {
                    new BulmaUtils(this).toast('Your submission was sent successfully!', 4000, "is-success");
                    this.submitting--;
                    this.$emit("close");

                    try {
                      const loggedin = await this.$store.dispatch(types.FETCH_CONTEST_DATA);
                      if(!loggedin) {
                        this.$router.push("/");
                      }
                    } catch (err) {
                      console.error(err);
                      new BulmaUtils(this).toast("Error contacting the server", 4000, "is-danger");
                    }
                }).catch((err) => {
                    this.submitting--;
                    if(err.status === 401 || err.status === 403) {
                        new BulmaUtils(this).toast("Not logged in, failed to submit!", 4000, "is-danger");
                        return this.$router.push("/");
                    }

                    if(!err.body.error) {
                      console.error(err);
                      new BulmaUtils(this).toast("Error contacting the server", 4000, "is-danger");
                    } else 
                      new BulmaUtils(this).toast(`Submission error: ${err.body.error}`, 4000, "is-danger");
                });
            },
            getMode() {
              return Helper.getBraceMode(this.language);
            }
        },
        components: {
          Brace
        }
    }
</script>

<style lang="sass"></style>